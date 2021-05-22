```bash
#!/bin/bash

FILES=$(find . -type f -iname "*.mov")

mkdir -p processed

for FILE in $FILES; do
    LC_FILE=$(echo "$FILE" | tr '[:upper:]' '[:lower:]')
    mv "$FILE" "$LC_FILE"

    MP4_FILE="${LC_FILE%%.mov}.mp4"

    if [[ ! -e "$MP4_FILE" ]]; then
        ffmpeg -i "$LC_FILE" "$MP4_FILE"
    fi

    mv "$LC_FILE" processed/
done
```
