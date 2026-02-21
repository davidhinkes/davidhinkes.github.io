# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a GitHub Pages site (`davidhinkes.github.io`) that publishes Markdown files directly. GitHub Pages renders the Markdown automatically — there is no local build step, no package manager, and no static site generator configuration.

## Structure

- `index.md` — Site landing page listing available courses
- `Tensors/` — A 14-lesson course, "Tensors and Deep Neural Networks," covering tensor operations from first principles through attention mechanisms

## Content conventions

- Course lessons in `Tensors/` follow the naming pattern `NN-Title-With-Dashes.md` (zero-padded two-digit prefix)
- Internal links omit the `.md` extension (e.g., `[Lesson](01-What-is-a-Tensor)`)
- New courses should get their own top-level directory with a `00-...-Index.md` index file, mirroring the `Tensors/` pattern
- The index file for a course should be added to `index.md`

## Deployment

Pushing to `master` deploys to `https://davidhinkes.github.io` via GitHub Pages automatically.
