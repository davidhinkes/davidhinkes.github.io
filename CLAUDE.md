# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a GitHub Pages site (`davidhinkes.github.io`) that publishes Markdown files directly. GitHub Pages renders the Markdown automatically — there is no local build step, no package manager, and no static site generator configuration.

## Structure

- `index.md` — Site landing page listing available courses
- `Tensors/` — 14-lesson course: "Tensors and Deep Neural Networks"
- `WebAssembly/` — 12-lesson course: "WebAssembly" (C and Go focused)

## Content conventions

- Course lessons follow the naming pattern `NN-Title-With-Dashes.md` (zero-padded two-digit prefix)
- Internal links omit the `.md` extension (e.g., `[Lesson](01-What-is-a-Tensor)`) — GitHub Pages requires this
- New courses get their own top-level directory with a `00-...-Index.md` index file and an entry in `index.md`

## Deployment

Pushing to `master` deploys to `https://davidhinkes.github.io` via GitHub Pages automatically.
