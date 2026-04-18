TEX  = xelatex
MAIN = main
FLAGS = -interaction=nonstopmode -halt-on-error

all: pdf

pdf:
	$(TEX) $(FLAGS) $(MAIN).tex
	$(TEX) $(FLAGS) $(MAIN).tex

watch:
	latexmk -pvc -xelatex $(MAIN).tex

clean:
	rm -f *.aux *.log *.nav *.out *.snm *.toc *.vrb *.fls *.fdb_latexmk *.synctex.gz

distclean: clean
	rm -f $(MAIN).pdf

.PHONY: all pdf watch clean distclean
