# CineCraft original PDF — lossless multipart storage

Source: https://www.cs.cornell.edu/abe/projects/cinecraft/cinecraft_chi2026.pdf
License: CC BY 4.0, author attribution retained inside original PDF.

The complete original bytes are retained across eight ordered binary parts. These are not individual PDF documents and cannot be opened independently. No compression, image alteration or text extraction was applied. One canonical source is counted, not eight.

From the repository root on macOS/Linux:

```sh
cat upstream/web__www.cs.cornell.edu/SRC-3c7032ec32ab/original.pdf.part00? > CineCraft.pdf
shasum -a 256 CineCraft.pdf
```

Expected length: 79078004 bytes. Expected SHA256: `62da9ef200233104e1d525b428839e0176f98e7ac38f402953fa90d0ced5a09c`.
The user ZIP copy contains the original complete PDF directly and needs no reconstruction.
