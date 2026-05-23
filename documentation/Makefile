# make file for documentation for transparent26

# assumes you have a working pandoc stack on the build machine...

PUBLISHED=\
  transparent-mode-implementation-guidance.md

all:	${PUBLISHED}

clean:
	rm -f ${PUBLISHED}

transparent-mode-implementation-guidance.pdf:	research/transparent-mode-implementation-guidance.md
	pandoc --toc -o transparent-mode-implementation-guidance.pdf research/transparent-mode-implementation-guidance.md

