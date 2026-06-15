CIRCDIR  := ./
OUTDIR   := ../out
IMGDIR   := ../assets/img/ckt

$(OUTDIR)/%.svg: $(CIRCDIR)/%.tex | $(OUTDIR)
	latexmk -f -pdf -output-directory=$(OUTDIR) $
	pdftoppm -r 600 -png -singlefile $(OUTDIR)/$*.pdf $(IMGDIR)/$*
	latexmk -C -output-directory=$(OUTDIR) $

$(OUTDIR):
	mkdir -p $(OUTDIR)

.PHONY: $(MAKECMDGOALS)
$(MAKECMDGOALS): $(OUTDIR)/$(MAKECMDGOALS).svg