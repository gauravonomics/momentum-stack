# Email Inbox

This directory stores raw email exports before processing.

**Immutability rule:** Files in this directory are never modified after creation. They are the original source of truth.

Ingestion processes read from this directory and write compiled, entity-linked email files to `wiki/communications/email/YYYY/MM/`.
