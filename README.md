# acadrust-layr

Source code of the modified [acadrust](https://github.com/hakanaktt/acadrust) 0.6.0 used by the LAYR iOS app to open DWG files, published under the Mozilla Public License 2.0.

LAYR（iOS App）用來開 DWG 圖檔的 acadrust 0.6.0 修改版原始碼，依 MPL-2.0 公開。

## Upstream 上游

- crates.io: `acadrust` 0.6.0
- `.crate` SHA-256: `d7cbc67281dc3c099c3f35c268dd80a04215ffa0580c37510d4fe7a6e4d6d14f`
- Upstream commit 上游提交: `9820fd3f51b7e31ca2fc95df0312011755816617`
- License 授權: MPL-2.0 (see [LICENSE](LICENSE)). The modified files stay under MPL-2.0. 修改過的檔案沿用 MPL-2.0。

## How to read this repository 怎麼看

- The first commit is the unmodified upstream package. The second commit adds the LAYR changes, so `git diff HEAD~1 HEAD` shows every change.
- 第一個提交是未修改的上游套件，第二個提交是 LAYR 的修改；比對兩個提交即可看到全部差異。
- Not included: `src/docs/OpenDesign_Specification_for_.dwg_files.pdf` (a reference document, not source code; available from upstream).
- 未收錄：上游 `src/docs/` 的 DWG 規格 PDF（參考文件，不是程式原始碼，可從上游取得）。

## What changed 改了什麼

Purpose: stop damaged DWG files from triggering huge up-front memory allocations or long reads past the end of the file. Only the reading side is changed; writing is untouched.

目的：讀到損壞的 DWG 時，避免一次預先配置巨量記憶體或長時間讀過檔尾。只改讀取端，寫入端不變。

- `src/io/dwg/dwg_stream_readers/bit_reader.rs`: stop after reading more than 64 KiB past the end; check remaining length in `read_bytes`; cap pre-allocation at 65,536 elements; cap zero-filled buffers at 64 MiB.
- `src/io/dwg/dwg_reader.rs`: cap AC21 decompression at 100 MB and sections at 256 MiB.
- Use the capped pre-allocation in:
  - `src/io/dwg/dwg_document_builder.rs`
  - `src/io/dwg/dwg_stream_readers/object_reader/associative.rs`
  - `src/io/dwg/dwg_stream_readers/object_reader/class_object.rs`
  - `src/io/dwg/dwg_stream_readers/object_reader/dgn_linestyle.rs`
  - `src/io/dwg/dwg_stream_readers/object_reader/dynamic_block.rs`
  - `src/io/dwg/dwg_stream_readers/object_reader/entities.rs`
  - `src/io/dwg/dwg_stream_readers/object_reader/field.rs`
  - `src/io/dwg/dwg_stream_readers/object_reader/mod.rs`
  - `src/io/dwg/dwg_stream_readers/object_reader/objects.rs`
  - `src/io/dwg/dwg_stream_readers/object_reader/tables.rs`
- `.gitattributes`: turn off text normalization so upstream CRLF bytes are kept.
- `LAYR-PATCHES.txt`: the same list in Traditional Chinese.

The caps limit pre-allocation only; they are not a total memory limit for the parser, and they are not a guarantee against every malicious file.

上限只限制預先配置，不是整個解析器的總記憶體上限，也不保證能擋下所有惡意檔案。
