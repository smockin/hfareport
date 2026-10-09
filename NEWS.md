# hfareport 0.1.0

* First release.
* `hfa_read_modules()` reads the HFA module exports from a folder or file paths.
* `hfa_clean()` builds the one-row-per-facility summary dataset.
* `hfa_generate_reports()` writes one colour-coded Word report per facility;
  `hfa_run()` runs the whole pipeline.
* Default template without a country emblem; `hfa_template_logo()` adds one.
* Configurable colour coding with `hfa_highlight_rules()` and
  `hfa_highlight_docx()`.
