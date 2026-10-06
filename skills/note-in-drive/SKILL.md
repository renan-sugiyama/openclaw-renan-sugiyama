---
name: note-in-drive
description: Saves a note to Google Drive as an editable document.
---

# Skill: note in Drive

## When to use
When I ask you to save a note, an idea, or a summary to Drive.

## Prerequisites

- The Zapier MCP server must be connected.
- The document conventions from AGENTS.md.

## Procedure

1. If the text was not provided, ask what should be saved.
2. Create the title using today's date and a short summary of the content.
3. Create the document using google_drive_create_file_from_text,
   with convert set to true.
4. Return the link.

## Expected output
A document exists in Drive whose name starts with AIE- and today's date,
and the agent returned its link.

## Special cases

- If the text is empty, do not create anything.