### Prepare
in ubuntu
```
sudo apt install tesseract-ocr
sudo apt install libtesseract-dev
sudo apt install tesseract-ocr-chi-sim #Support Chinese
```

### search

#### one line command 
```
# search *.pig in current directory
# convert image colorspace to gray
# identify text in image
for f in `find . -iname *.png`;do echo $f;convert -colorspace Gray $f $f; tesseract $f stdout --oem 1 --psm 11 >$f.search; done

find . -iname *.search |xargs grep -i xxxx  # search xxx in identified texts
```
