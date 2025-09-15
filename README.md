package main

import (
	"flag"
	"fmt"
	"image"
	"os"
	"path/filepath"
	"strings"

	"github.com/disintegration/imaging"
)

func main() {
	in := flag.String("in", "", "input image file (required)")
	out := flag.String("out", "", "output file (optional, will infer ext)")
	width := flag.Int("w", 0, "resize width (optional)")
	height := flag.Int("h", 0, "resize height (optional)")
	quality := flag.Int("q", 95, "jpeg quality")
	flag.Parse()

	if *in == "" {
		fmt.Println("Usage: -in input.jpg [-out out.png] [-w 800] [-h 600]")
		return
	}
	src, err := imaging.Open(*in)
	if err != nil {
		fmt.Println("open error:", err)
		return
	}
	dst := src
	if *width > 0 || *height > 0 {
		dst = imaging.Resize(src, *width, *height, imaging.Lanczos)
	}
	outPath := *out
	if outPath == "" {
		ext := strings.ToLower(filepath.Ext(*in))
		outPath = strings.TrimSuffix(*in, ext) + "_converted" + ext
	}
	ext := strings.ToLower(filepath.Ext(outPath))
	f, err := os.Create(outPath)
	if err != nil {
		fmt.Println("create error:", err)
		return
	}
	defer f.Close()

	switch ext {
	case ".jpg", ".jpeg":
		err = imaging.Encode(f, dst, imaging.JPEG, imaging.JPEGQuality(*quality))
	case ".png":
		err = imaging.Encode(f, dst, imaging.PNG)
	case ".gif":
		err = imaging.Encode(f, dst, imaging.GIF)
	case ".tif", ".tiff":
		err = imaging.Encode(f, dst, imaging.TIFF)
	default:
		// infer format from file content
		err = imaging.Encode(f, dst, imaging.PNG)
	}
	if err != nil {
		fmt.Println("encode error:", err)
		return
	}
	fmt.Println("Written:", outPath, " Size:", dst.Bounds().Dx(), "x", dst.Bounds().Dy())
}
