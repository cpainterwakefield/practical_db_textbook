To build .svg files from .tex files:

First, install:
   - the TeXLive system (may need some extra packages)
   - pdf2svg
   - librsvg2-bin (for rsvg-convert)
   - Python 'scour' package

If you have Gnu Make installed, just run 'make' in this directory.
Else,
1. Run 'pdflatex <this-filename>' in this directory
2. Run 'pdf2svg <this-filename> temp.svg'
3. Run 'rsvg-convert temp.svg -o temp2.svg -z 2 -f svg'
4. Run 'scour -i temp2.svg -o <this-filename-base>.svg' (this just makes a
   smaller svg file)
5. (Optional) clean up unneeded files:
      'rm -f *.log *.aux <this-filename-base>.pdf temp1.svg temp2.svg'
