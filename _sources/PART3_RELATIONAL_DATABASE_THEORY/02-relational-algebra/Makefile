SOURCES=tree1.tex tree2.tex

all: $(SOURCES:.tex=.svg)

%.svg: %.temp2
	scour -i $< -o $@

%.temp2: %.temp1
	rsvg-convert $< -o $@ -z 1.25 -f svg

%.temp1: %.pdf
	pdf2svg $< $@

%.pdf: %.tex
	pdflatex $<
	rm -f *.aux *.log

clean: mostlyclean
	rm -f $(SOURCES:.tex=.svg)

# Everything except the final SVG files
mostlyclean:
	rm -f *.aux *.log *.temp1 *.temp2 $(SOURCES:.tex=.pdf)
