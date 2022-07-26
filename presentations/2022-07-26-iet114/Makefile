slides-for-meetecho.pdf: slides.pdf
	# Workaround for meetecho doing slides at fixed DPI rather than fixed
	# size in pixels
	#
	# Thanks Kurt Pfeifle at https://stackoverflow.com/a/7507511
	gs \
	 -o $@ \
	 -sDEVICE=pdfwrite \
	 -g16000x9000 \
	 -dFIXEDMEDIA \
	 -dPDFFitPage \
	 -dCompatibilityLevel=1.4 \
	  $<

slides.pdf: slides.tex
	lualatex $<
