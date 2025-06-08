# Docker Multi-Stage Build Example

This project demonstrates how to use Docker's multi-stage build feature to create a lightweight image by building a test app (in this case, the latest version of Vim 9) then copying only the necessary files into a final image.

## Example Dockerfile

```dockerfile
# Stage 1: Build Vim from source
FROM alpine:latest AS builder
WORKDIR /usr/src
RUN apk add clang \
            gcc \
            git \
            ncurses-dev \
            make \
            bash
RUN git clone --depth 1 https://github.com/vim/vim.git
WORKDIR /usr/src/vim
RUN ./configure
RUN make -j "$(nproc)"
RUN make install

# Stage 2: Create final image with Vim and dependencies
FROM alpine:latest
# Vim won't run without tput (ncurses)
RUN apk add bash ncurses
COPY --from=builder /usr/local/bin/vim /usr/local/bin/vim
COPY --from=builder /usr/local/bin/vimtutor /usr/local/bin/vimtutor
COPY --from=builder /usr/local/bin/xxd /usr/local/bin/xxd
COPY --from=builder /usr/local/share/vim /usr/local/share/vim
COPY --from=builder /usr/local/share/man/man1/evim.1 /usr/local/share/man/man1/evim.1
COPY --from=builder /usr/local/share/man/man1/vim.1 /usr/local/share/man/man1/vim.1
COPY --from=builder /usr/local/share/man/man1/vimdiff.1 /usr/local/share/man/man1/vimdiff.1
COPY --from=builder /usr/local/share/man/man1/vimtutor.1 /usr/local/share/man/man1/vimtutor.1
COPY --from=builder /usr/local/share/man/man1/xxd.1 /usr/local/share/man/man1/xxd.1
WORKDIR /usr/local/bin
RUN ln -s vim ex
RUN ln -s vim rview
RUN ln -s vim rvim
RUN ln -s vim view
RUN ln -s vim vimdiff
WORKDIR /usr/local/share/man/man1
RUN ln -s vim.1 ex.1
RUN ln -s vim.1 rview.1
RUN ln -s vim.1 rvim.1
RUN ln -s vim.1 view.1
WORKDIR /root
```
