# Build the deck. Drop this file into any deck directory as-is.
#
#   make          build presentation.pdf
#   make watch    rebuild automatically on save
#   make view     build, then open the pdf
#   make clean    remove build artifacts, keep the pdf
#   make distclean  also remove the pdf

DOC     := presentation
LATEXMK := latexmk
FLAGS   := -pdf -interaction=nonstopmode -halt-on-error
DEPS    := $(DOC).tex beamerthemecookie.sty $(wildcard assets/*)

.PHONY: all watch view clean distclean
.DEFAULT_GOAL := all

all: $(DOC).pdf

$(DOC).pdf: $(DEPS)
	$(LATEXMK) $(FLAGS) $(DOC).tex
	@boxes=$$(grep -c Overfull $(DOC).log || true); \
	 if [ "$$boxes" -gt 0 ]; then echo "warning: $$boxes overfull box(es) -- see $(DOC).log"; fi

watch:
	$(LATEXMK) $(FLAGS) -pvc $(DOC).tex

view: $(DOC).pdf
	@(xdg-open $(DOC).pdf >/dev/null 2>&1 &) || open $(DOC).pdf

clean:
	$(LATEXMK) -c $(DOC).tex
	rm -f $(DOC).nav $(DOC).snm $(DOC).vrb $(DOC).listing

distclean: clean
	rm -f $(DOC).pdf
