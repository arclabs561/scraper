VERSION 0.7
FROM scratch
ARG --global VERSIONS_SEP="  "
ARG --global BUILD_DIR=./build

lint:
	WAIT
		DO +LINT --target=+lint-go
		DO +LINT --target=+lint-py
		DO +LINT --target=+lint-other
	END

LINT:
	COMMAND
	ARG --required target
	BUILD $target
	COPY $target/VERSIONS /artifacts/VERSIONS$target
	COPY $target/OUTPUT /artifacts/OUTPUT$target
	SAVE ARTIFACT /artifacts/* AS LOCAL $BUILD_DIR/artifacts/

lint-go:
	FROM golang:1.21
	RUN go install github.com/google/yamlfmt/cmd/yamlfmt@latest
	RUN go install github.com/rhysd/actionlint/cmd/actionlint@latest
	ARG GOLANGCI_LINT_CACHE=/.cache/golangci-lint
	CACHE $GOLANGCI_LINT_CACHE
	RUN go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest
	RUN for tool in $(ls $GOPATH/bin); do \
	  echo "$tool$VERSIONS_SEP$(go version -m $(which $tool) | awk '$1=="mod" {print $3}')" | tee -a VERSIONS; \
	done
	SAVE ARTIFACT VERSIONS
	DO +JUST --recipe=lint-go

lint-py:
	FROM python:3.11
	RUN python3 -m pip install yamllint
	RUN yamllint --version | awk -v sep="$VERSIONS_SEP" '{printf "%s"sep"%s\n", $1, $2}' | tee -a VERSIONS
	SAVE ARTIFACT VERSIONS
	DO +JUST --recipe=lint-py

lint-other:
	FROM debian:12
	RUN touch VERSIONS
	SAVE ARTIFACT VERSIONS
	DO +JUST --recipe=lint-other

JUST:
	COMMAND
	ARG --required recipe
	COPY +lint-dep-just/just /usr/local/bin/
	COPY . /src
	WORKDIR /src
	RUN just $recipe | tee OUTPUT
	RUN cat OUTPUT
	SAVE ARTIFACT OUTPUT


lint-dep-just:
	FROM rust:1.71
	RUN cargo install just
	RUN just --version
	SAVE ARTIFACT /usr/local/cargo/bin/just
