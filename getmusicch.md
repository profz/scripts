#!/bin/bash

# Prompt for a search query
query=$(fzf --prompt="Search music: " --print-query --no-select-1 --no-sort --height=10)
[[ -z "$query" ]] && exit 1

# Search and parse using yt-dlp JSON for fast, clean data
mapfile -t results < <(yt-dlp "ytsearch20:$query" --skip-download --print-json | jq -r '[.title, .uploader, .duration_string, .webpage_url] | @tsv')

# Show selection menu
selected=$(printf "%s\n" "${results[@]}" | fzf --with-nth=1,2,3 --delimiter=$'\t' --preview='echo {}' --height=20)
[[ -z "$selected" ]] && exit 1

# Extract the URL
url=$(awk -F'\t' '{print $4}' <<< "$selected")
[[ -z "$url" ]] && exit 1

# Download with metadata and cover art
yt-dlp "$url" \
  --extract-audio \
  --audio-format mp3 \
  --embed-thumbnail \
  --embed-metadata \
  --no-playlist \
  -o "$HOME/Music/%(title)s.%(ext)s"

notify-send "Download complete" "$(yt-dlp --get-title "$url")"

