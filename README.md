# Oracle Analytics Custom Visualization: Correlation Matrix (com-oraclecic-correlationmatrix)

Correlation Matrix custom visualization plugin for Oracle Analytics (OAC/OAS). It computes pairwise correlations between selected numeric measures and renders an interactive heatmap using Plotly, with optional Pearson or Spearman correlation and visual significance markers.

- Correlation types: Pearson, Spearman (toggle via dropdown)
- Significance overlay: X markers where |r| is statistically significant at the configured confidence level (default 95%)
- Value labels: Cell text shows r rounded to 2 decimals and indicates “(p < 0.05)” when significant
- Responsive layout and OAC theming-compatible fonts
- Localization scaffold present (NLS bundles)
- No server-side code required

Version: 1.0.0

## Demo/Preview

The chart renders a heatmap correlation matrix with a vertical color bar:
- Colorscale: red (-1) → neutral (~0) → green (+1)
- X markers on cells that pass the significance threshold
- Dropdown to switch between Pearson and Spearman

## Data Requirements

Based on the Data Model Handler configuration:
- Rows (categorical): exactly 1 field required
- Measures (numeric): between 2 and 5 measures required
- Other placements (color, detail, glyph, size): not used
- Measure label: hidden by default

Additional runtime behavior:
- Columns with fewer than 2 valid numeric, non-null, non-NaN and non-zero values are dropped from correlation computation
- Pairwise computation ignores null/NaN values on a per-pair basis

Data governor (soft limits from plugin properties):
- Rows: up to 10,000
- Columns: up to 5,000
These are upper bounds; performance depends on the number of measures and rows you actually load.

## Installation

Prerequisites:
- Oracle Analytics Cloud (OAC) or Oracle Analytics Server (OAS) with permissions to import custom plugins
- Custom Viz enabled in your tenant/instance

1) Package the plugin
- Ensure this project’s directory structure is preserved exactly
- Create a zip archive of the plugin folder contents (not nesting multiple levels)
  - Windows (File Explorer): Right-click the `com-oraclecic-correlationmatrix` folder → Send to → Compressed (zipped) folder
  - PowerShell example:
    ```
    # Run in the parent directory of com-oraclecic-correlationmatrix
    Compress-Archive -Path .\com-oraclecic-correlationmatrix\* -DestinationPath com-oraclecic-correlationmatrix.zip -Force
    ```
  - macOS/Linux example:
    ```
    cd com-oraclecic-correlationmatrix
    zip -r ../com-oraclecic-correlationmatrix.zip .
    ```

2) Import into OAC
- Open OAC → Console/Administration → Extensions (or Data Visualization Console → Extensions)
- Go to Visualizations
- Click Import (or Add) and select `com-oraclecic-correlationmatrix.zip`
- Confirm to install; the visualization should appear under its category

3) Verify
- Create or open a Workbook
- Add the “Correlationmatrix Plugin” visualization from the gallery
- Place 1 categorical column on Rows and 2–5 measures on Values

If the visualization does not appear after import, refresh the browser and re-open OAC.

## Usage

1) Add the visualization to your canvas
2) Drag one categorical attribute into Rows to define the observation set
3) Drag 2–5 numeric measures (columns) into Values
4) Use the top-right dropdown to switch correlation type:
   - Pearson: linear correlation of centered values
   - Spearman: Pearson correlation computed on ranked values
5) Resize the tile as needed; the chart is responsive
6) Interpretation:
   - Heatmap cell color encodes correlation coefficient r in [-1, 1]
   - “X” marks denote cells significant at the configured confidence level (default 95%)
   - Tooltip/labels display r rounded to 2 decimals, with “(p < 0.05)” when significant

Notes:
- Significance uses a two-tailed Pearson critical values table with linear interpolation by degrees of freedom df = n − 2, where n is the number of valid pairs for that measure pair.
- When using Spearman, significance markers still use the Pearson critical value approximation (a practical heuristic).

## Project Structure

- correlationmatrix.js
  - Main AMD module, registers the visualization and renders Plotly heatmap
  - Provides Pearson/Spearman toggle and significance overlay
- correlationmatrixdatamodelhandler.js
  - Declares the logical-to-physical mapping and placement constraints (Rows, Measures)
- utils/correlationUtils.js
  - Numeric helpers: Pearson, Spearman (via ranks), significance matrix using critical values tables and interpolation
- correlationmatrixstyles.css
  - Minimal CSS for container sizing and responsiveness
- plotly.min.js
  - Bundled Plotly runtime used by the visualization
- nls/
  - messages.js and root/messages.js for i18n scaffold
- extensions/
  - oracle.bi.tech.plugin.visualization/com.oraclecic.correlationmatrix.json
    - Host and visualization metadata (method: createClientComponent, module id)
  - oracle.bi.tech.plugin.visualizationDatamodelHandler/com.oraclecic.correlationmatrix.visualizationDatamodelHandler.json
    - Data model handler module and placement constraints
- correlationmatrixIcon.png
  - Icon asset referenced by the visualization manifest

Module IDs and requirejs:
- Root module id prefix: `com-oraclecic-correlationmatrix`
- Manifest host.script.method: `createClientComponent`
- Visualization module: `com-oraclecic-correlationmatrix/correlationmatrix`
- Plotly exposed via requirejs path `Plotly: com-oraclecic-correlationmatrix/plotly.min`

## How It Works (Algorithms)

- Pearson correlation:
  - Center each series (subtract mean), compute r = cov(X, Y) / (σX σY)
  - Rounded to 4 decimals internally, displayed to 2 decimals in the heatmap text
- Spearman correlation:
  - Rank transform of each series with average ranks for ties, then apply Pearson on ranks
- Significance:
  - Degrees of freedom: df = n − 2 (n = number of valid (x, y) pairs)
  - Critical values table for 80/90/95/98/99% is embedded (utils/CorrelationUtilities)
  - Linear interpolation between table entries by df when exact df not found
  - A cell is “significant” if |r| ≥ critical_value(df, confidence)
  - The UI displays X markers on significant cells and “(p < 0.05)” in text at 95% confidence

Caveats:
- The significance approximation uses the Pearson table even when using Spearman; for large samples this is often acceptable as a heuristic.
- Columns with ≤1 valid non-zero numeric value are excluded from correlation entirely.

## Development

Requirements:
- No build system required; the plugin uses AMD modules bundled as static assets
- Ensure module ids and paths match the manifests

Common tasks:
- Bump version:
  - Update `Correlationmatrix.VERSION` in correlationmatrix.js
  - Optionally update `_version` in `extensions/.../com.oraclecic.correlationmatrix.json`
- Adjust styling:
  - Edit `correlationmatrixstyles.css`
- Modify algorithms:
  - Edit `utils/correlationUtils.js` (Pearson, Spearman, significance)
- Localization:
  - Add keys to `nls/root/messages.js` and provide translated bundles under `nls/<locale>/messages.js`
  - The manifests reference keys like `CORRELATIONMATRIX_DISPLAY_NAME`, etc.

Testing in OAC:
- Zip and re-import the plugin to test changes
- If OAC caches assets, remove the old plugin first or bump version, then import

## Troubleshooting

- The visualization does not appear in the gallery:
  - Verify the zip contains the plugin files at the root (no extra nesting)
  - Confirm manifests exist and paths match:
    - Visualization JSON: module `com-oraclecic-correlationmatrix/correlationmatrix`
    - Data model handler JSON: module `com-oraclecic-correlationmatrix/correlationmatrixdatamodelhandler`
  - Ensure the plugin id remains unique in your tenant
- Blank/empty chart:
  - Ensure 1 categorical attribute is placed on Rows
  - Ensure 2–5 numeric measures are placed on Values
  - Check that each measure has more than 1 valid non-zero numeric value
- Significance markers missing:
  - With very small n, the critical value can be high; many cells may not pass the threshold
  - df = n − 2; verify you have enough valid rows
- Performance issues:
  - Large row counts increase computation time; if necessary, filter/summarize data
  - Reduce the number of measures to reduce matrix size

## Packaging and Deployment Tips

- Keep the directory name and module ids consistent with the manifests
- Prefer re-import after changes; clear browser cache if assets appear stale
- To enable in Oracle Analytics:
  - Admin Console → Extensions → Visualizations → Import
  - Assign permissions to users/groups if your environment requires it

## Security and Dependencies

- Plotly is bundled as `plotly.min.js` and loaded via AMD
- No external network calls are made
- The plugin executes fully client-side within the OAC workbook context

## Roadmap / Ideas

- User-configurable confidence level (80/90/95/98/99)
- Tooltips with additional stats (n, df, p-value approximation)
- Color scale customization and dark theme presets
- Option to annotate only upper/lower triangle

## Changelog

- 1.0.0
  - Initial release with Pearson/Spearman, significance overlay, responsive heatmap

## Contributing

Pull requests and issues are welcome. Please include:
- A clear description of the problem or enhancement
- Reproduction steps or sample data
- Proposed changes with rationale

## License

Specify your project license here (e.g., MIT, Apache-2.0). If you need a template, you can add a LICENSE file at the project root.

## Acknowledgements

- Plotly (plotly.js) for heatmap rendering
- Oracle Analytics Custom Visualization framework
