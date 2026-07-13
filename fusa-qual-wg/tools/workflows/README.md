# Workflows

### How to Generate Graphs:
1. Create your Mermaid graph in this directory and save it with a `.mmd` extension.
2. Run the `graph_generator.py` script.

The resulting graph images will be saved in the `generated_graph` folder using the same base filenames.

### Usage Examples

**1. Generate all graphs in the directory (Default)** If you run the script without any arguments, it will automatically find and process every `.mmd` file in this folder.
```bash
python graph_generator.py
```

**2. Generate specific graph** Pass the exact filename as an argument if you only want to process one file:
```bash
python graph_generator.py verify_tool_evidence_requirements.mmd
```

**3. Generate multiple specific graphs** You can process several specific files at once by separating their names with a space:
```bash
python graph_generator.py file1.mmd file2.mmd file3.mmd
```
