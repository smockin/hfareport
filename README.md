# hfareport

<!-- badges: start -->
[![R-CMD-check](https://github.com/smockin/hfareport/actions/workflows/R-CMD-check.yaml/badge.svg)](https://github.com/smockin/hfareport/actions/workflows/R-CMD-check.yaml)
<!-- badges: end -->

Health Facility Assessment (HFA) summary reports, from REDCap exports to
colour-coded Word documents, in R.

`hfareport` takes the module exports of a neonatal Health Facility Assessment
collected in REDCap. It builds a one-row-per-facility summary dataset, then
writes **one editable Word report per facility**. Cells that signal a gap
(stock-outs, missing or broken equipment, overcrowding, missing guidelines)
are shaded so they stand out.

## Installation

```r
# install.packages("remotes")
remotes::install_github("smockin/hfareport", build_vignettes = TRUE)
```

## Quick start

Put the CSV exports of the HFA modules in one folder, then:

```r
library(hfareport)

hfa_run("path/to/rawdata", output_dir = "reports")
```

This writes one `FacilityID_<id>_<date>_HFA_Summary_Report.docx` per facility
in `reports/`. To try it on the synthetic example data shipped with the
package:

```r
hfa_run(hfa_example("raw"), output_dir = tempdir())
```

The full walkthrough is in the vignette:

```r
vignette("hfareport")
```

## The three steps

```r
raw  <- hfa_read_modules("path/to/rawdata")       # 1. read the exports
summ <- hfa_clean(raw)                            # 2. build the summary dataset
hfa_generate_reports(summ, output_dir = "reports")  # 3. write the reports
```

| Function | Purpose |
|----------|---------|
| `hfa_read_modules()` | Read the module CSV exports, from a folder or from individual file paths |
| `hfa_clean()` | Merge, decode, apply branching logic and derive the report indicators |
| `hfa_save()` | Save the summary dataset as .csv or .rds |
| `hfa_generate_reports()` | Fill the Word template and colour-code it, one report per facility |
| `hfa_run()` | All three steps in one call |
| `hfa_highlight_rules()` | Set the colour-coding rules |
| `hfa_highlight_docx()` | Colour-code any filled report again, e.g. after manual edits |
| `hfa_check_template()` | Check a template and keyword file against the data |
| `hfa_template_logo()` | Put a country emblem or logo into the default template |
| `hfa_keywords()`, `hfa_bookmarks()` | The placeholder and bookmark maps used to fill the template |
| `hfa_example()` | Paths to the bundled template, keyword file and example data |

## Input: the HFA modules

Export each instrument from REDCap as CSV with **raw (coded) values**. Files
are matched to modules by their **column names**, so they can be named
anything and passed in any order.

| Key | Module | Identified by |
|-----|--------|---------------|
| fac | Facility infrastructure | `inf_id_dtfrm` |
| neo | Neonatal unit infrastructure | `inf_id2_dtfrm` |
| pha | Laboratory and pharmacy | `lab_id_dtfrm` |
| med | Medical devices | `md_id_dtfrm` |
| bio | Biomedical engineering / workshop | `ws_id_dtfrm` |
| hum | Human resources | `hr_id_dtfrm` |
| inf | Information systems | `is_id_dtfrm` |
| gov | Governance | `gv_id_dtfrm` |
| fam | Family-centred care | `fcc_id_dtfrm` |
| obs | Hand hygiene observation | `obs_id_dtfrm` |

The facility module (`fac`) is required. Any other module that is missing
gives a warning, and its report fields show as not recorded. The hand hygiene
observation module is read but not used in the summary report.

Dates are read in whichever format the export uses (`2026-09-15`, or
`15/9/2026` / `9/15/2026` after a file has been re-saved in Excel). The
format is detected once across all date columns; set
`hfa_clean(date_format = "dmy")` to force one.

## Output: the reports

* One `.docx` per facility ID in the summary dataset (or in `facility_ids`).
* Written to `output_dir`, which defaults to the working directory.
* Ordinary Word documents, so they can be edited before they are shared.
* Missing values appear as `N/R` (not recorded); `N/A` means not applicable.

## Templates and keywords

The report template is a Word document that contains short placeholder codes
(keywords, e.g. `kcvqt9`) where values go, plus a few bookmarks for the
header and the introductory lines. A keyword file maps each code to a
variable in the summary dataset.

* **Default:** the package ships a template with no country emblem, and the
  matching keyword file. Nothing needs to be supplied.
* **Your own:** pass `template =` and `keywords =`. Keyword files may use the
  columns `variable, keyword`, or `bkm, data, keyword`.

```r
hfa_check_template("My_Template.docx", "my_keywords.csv", data = summ)

hfa_generate_reports(summ,
  template = "My_Template.docx",
  keywords = "my_keywords.csv")
```

To add a country emblem or organisation logo (PNG) to the default template:

```r
hfa_template_logo("emblem.png", "My_HFA_Template.docx")
hfa_generate_reports(summ, template = "My_HFA_Template.docx")
```

## Colour coding

Each rule is an argument of `hfa_highlight_rules()`:

```r
rules <- hfa_highlight_rules(colour = "FFC7CE", bed_pc_threshold = 90)
hfa_generate_reports(summ, rules = rules)

hfa_generate_reports(summ, highlight = FALSE)               # plain reports
hfa_highlight_docx("edited.docx", "edited_coloured.docx")   # re-colour an edited report
```

By default the following are shaded:

* "No", "Not available", "0 Available" and stock-outs
* devices with none functional ("0/n Functional")
* more than one baby per cot "Frequently" or "Sometimes"
* more than one baby per radiant warmer/incubator at any frequency other than "Never"
* bed occupancy above 95%
* zero babies, zero occupancy and zero KMC beds or chairs
* "Some" for solar power or battery inverter functionality
* sinks not all functioning
* no data clerk assigned

## Country-specific adjustments

```r
hfa_clean(raw,
  drop_ids      = c(310, 311, 313),                      # remove test records
  duplicate_ids = data.frame(module = "fac",             # sites that share a
                             from = 401, to = 407),      # module record
  overrides     = data.frame(id_recidfac = 128,          # correct single cells
                             variable = "inf_facid_typ",
                             value = "District Hospital"))
```

## Acknowledgements

The data-cleaning logic in `hfa_clean()` is an R translation of the Stata
code for the HFA summary dataset developed by **Rebecca Penzias** and
**Eric Ohuma**. Their work defined the variable derivations, branching-logic
rules and report indicators that this package reproduces. Both are listed as
contributors in the package `DESCRIPTION`.

## Citation

```r
citation("hfareport")
```

## Licence

MIT © Morris Ogero. See `LICENSE`.
