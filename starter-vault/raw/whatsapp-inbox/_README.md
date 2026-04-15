# WhatsApp Inbox

This directory stores raw WhatsApp message exports (text and screenshot captures) before processing.

**Immutability rule:** Files in this directory are never modified after creation. They are the original source of truth.

Ingestion processes read from this directory and write compiled, entity-linked message files to `wiki/communications/whatsapp/YYYY/MM/`.
