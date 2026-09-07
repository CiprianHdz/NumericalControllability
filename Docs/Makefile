MAIN := theory

.PHONY: all clean

all: $(MAIN).pdf

$(MAIN).pdf: $(MAIN).tex $(MAIN).bib $(wildcard sections/*.tex)
	latexmk -pdf -interaction=nonstopmode $(MAIN).tex

clean:
	latexmk -C $(MAIN).tex
	rm -f *.bbl *.blg *.run.xml *-blx.bib
