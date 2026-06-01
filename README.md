<div align="center">
  <h1>
    <img src="images/medsimlogo.png" alt="MedSim Logo" width="40" style="vertical-align: middle; margin-right: 10px;"/>
    MedSim: High-Fidelity Clinical Simulator
  </h1>
  <p><i>An AI-powered medical simulation ecosystem secured by RAG and evaluated via MLOps benchmarking.</i></p>
</div>

---

## 🎥 Project Demonstration

Before diving into the code, watch our full system demonstration showcasing the dynamic VRAM swapping and our customizable Multi-Agent Engine, allowing you to seamlessly switch between 5 different LLMs (Llama-3, Gemma-2, Mistral, BioMistral, and Phi-3) for both the RAG-augmented Patient and the Virtual Professor:

<div align="center">
  <a href="https://www.youtube.com/watch?v=sl_wyVNGyYw">
    <img src="https://img.youtube.com/vi/sl_wyVNGyYw/maxresdefault.jpg" alt="MedSim Demo Video" width="800" style="border-radius: 10px; box-shadow: 0 4px 8px rgba(0,0,0,0.2);"/>
  </a>
  <br>
  <p><i>👉 Click the image above to watch the MedSim Demo on YouTube 👈</i></p>
</div>

---

## 📖 Overview

Clinical interviewing is a cornerstone of medical practice, yet students lack a "safe space" to make clinical errors and receive immediate expert feedback. **MedSim** solves this by providing a dual-agent simulation environment:

1. **The Patient Agent:** A RAG-augmented LLM that simulates realistic symptoms based on validated medical literature, stripped of medical jargon.
2. **The Supervisor Agent (LLM-as-a-Judge):** A highly analytical evaluator that grades the student's clinical reasoning, diagnostic accuracy, and patient safety protocols.

This project was developed as part of the **INF 3600 - Generative Artificial Intelligence** course at **UiT The Arctic University of Norway**.

---

## 🔬 Research & Metrology: The Tri-Agent Sandbox

A massive portion of the workload for this project was dedicated to rigorous MLOps benchmarking. Validating a medical AI system manually is methodologically flawed and impossible to scale. To solve this, we engineered a fully automated **Tri-Agent Sandbox**.

Before deploying the dual-agent application for human users, we replaced the human medical student with an automated **"Doctor Agent"**. By deploying a combinatorial evaluation matrix across 5 different open-weight models (acting interchangeably as the Patient, Doctor, and Judge), we generated and evaluated extensive automated clinical interactions.

This intensive research phase was crucial to:
* **Benchmark Reasoning Limits:** Evaluate model performances across varying parameter sizes (from 3.8B to 9B) on complex medical traps.
* **Expose Critical LLM Flaws:** Detect and analyze behaviors such as compliance bias (sycophancy), persona drift, and medical hallucinations.
* **Determine Optimal Pairings:** Mathematically identify the safest and most rigorous model combinations to deploy in the final production UI.

---

## 🚀 Installation & Setup

### 1. Environment Preparation

We highly recommend using a dedicated Conda environment to avoid dependency conflicts, especially regarding PyTorch and CUDA versions.

```bash
conda create -n medsim_env python=3.11
conda activate medsim_env
```

### 2. Install Dependencies

Install the required packages. The `requirements.txt` is pre-configured to download the PyTorch version compatible with CUDA 12.1 (optimized for V100/Sigma2 nodes).

```bash
pip install -r requirements.txt
```

---

## ⚙️ How to Use MedSim

### Phase 1: Data Engineering (ETL)

To respect GitHub file size limits and copyright distributions, the full database of 4,500+ pathologies is **not included** in this repository.
To build the complete RAG database locally, run the scraper. It will use `pubmed-statpearls-set.txt` as a seed to query the NCBI database.

```bash
python etl/web_scrapping.py
```

*⚠️ **Note:** This process respects server rate limits and will take several hours to complete. For immediate testing, the repository includes a lightweight `knowledge_base_extract.json`.*

### Phase 2: The Interactive UI (Streamlit)

To launch the interactive clinical simulator designed for human medical students:

```bash
streamlit run app.py
```

This boots the dual-agent environment, featuring dynamic VRAM swapping to fit heavily quantized 7B-9B parameter models onto a single 16GB GPU.

### Phase 3: The MLOps Evaluation Factory

To objectively determine which LLM makes the best pedagogical judge, the system includes a fully automated benchmarking pipeline. It evaluates models like Meta Llama-3, Google Gemma-2, Mistral, BioMistral, and Phi-3.

Run the flagship algorithms in the following order:

1. **Generate the interactions:**
```bash
python scripts/run_benchmark.py
```

2. **Extract and aggregate the JSON data into a CSV:**
```bash
python scripts/extract_results.py
```

3. **Generate metrological visualizations:**
```bash
python scripts/generate_graphs.py
```

You can view the final metrics, including sycophancy bias and clinical reasoning variance, inside the `/results/graphs/` directory.

### Phase 4: The Doctor × Patient Combinatorial Benchmark
 
To go further, we extended the evaluation factory with a fully automated **Tri-Agent Sandbox**: instead of a human student, an LLM also plays the Doctor role. This allows systematic evaluation of all 25 possible Patient × Doctor model combinations (5 models × 5 models) without any human involvement, scored by a fixed Llama-3 judge.
 
Each simulation runs a multi-turn clinical interview where the Doctor agent asks diagnostic questions, the Patient agent responds based on RAG-injected symptoms, and the Judge scores both agents independently — using both heuristic rules (jargon detection, role leakage, response length) and semantic LLM evaluation.
 
Run the pipeline in the following order:
 
1. **Generate all 25 Patient × Doctor interactions:**
```bash
# Run all combinations
python scripts/run_benchmark_dr_pa.py --configs all
 
# Run a specific range
python scripts/run_benchmark_dr_pa.py --configs 1-5
 
# Run specific combinations by index
python scripts/run_benchmark_dr_pa.py --configs 1,3,7,12
 
# List all available configurations
python scripts/run_benchmark_dr_pa.py --list
 
# Skip LLM judge (heuristic scoring only, much faster)
python scripts/run_benchmark_dr_pa.py --configs all --no-judge
```
 
2. **Extract and aggregate the JSON transcripts into a CSV:**
```bash
python scripts/extract_benchmark_results_dr_pa.py
```
 
3. **Generate metrological visualizations:**
```bash
python scripts/generate_graphs_dr_pa.py
```
 
You can view the final metrics inside the `/results/graphs_simulation/` directory, including a 5×5 heatmap of all model pairings, role-specific score distributions, and a heuristic vs. LLM judge comparison.

Raw transcripts are saved as JSON files in the following structure:
```
results/benchmark_simulation/<patient_model>/patient_<X>_doctor_<Y>/<pathology>/transcript.json
```

---

## 🗂️ Pipeline Data Flow
 
The table below documents the input/output relationships of every script in the pipeline, making it easier to understand the architecture and debug data dependencies.
 
> **Legend:** `IN` = file read as input · `OUT` = file generated · `DEP` = internal module dependency · `EXT` = external service
 
<style>
:root {
  --bg-primary: #ffffff;
  --bg-secondary: #f5f5f3;
  --bg-tertiary: #eceae4;
  --text-primary: #1a1a18;
  --text-secondary: #5f5e5a;
  --border: rgba(0,0,0,0.12);
  --border-strong: rgba(0,0,0,0.22);
  --info-bg: #e6f1fb; --info-text: #0c447c;
  --success-bg: #eaf3de; --success-text: #3b6d11;
  --warn-bg: #faeeda; --warn-text: #854f0b;
  --danger-bg: #fcebeb; --danger-text: #a32d2d;
}
@media (prefers-color-scheme: dark) {
  :root {
    --bg-primary: #1e1e1c;
    --bg-secondary: #2c2c2a;
    --bg-tertiary: #242422;
    --text-primary: #f0ede6;
    --text-secondary: #b4b2a9;
    --border: rgba(255,255,255,0.10);
    --border-strong: rgba(255,255,255,0.20);
    --info-bg: #042c53; --info-text: #b5d4f4;
    --success-bg: #173404; --success-text: #c0dd97;
    --warn-bg: #412402; --warn-text: #fac775;
    --danger-bg: #501313; --danger-text: #f7c1c1;
  }
}
* { box-sizing: border-box; margin: 0; padding: 0; }
body { background: transparent; font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif; font-size: 13px; color: var(--text-primary); padding: 0; }
.pill { display: inline-block; border-radius: 4px; padding: 2px 7px; font-size: 11px; font-weight: 500; margin: 1px 2px; }
.pill-in  { background: var(--info-bg);    color: var(--info-text); }
.pill-out { background: var(--success-bg); color: var(--success-text); }
.pill-dep { background: var(--warn-bg);    color: var(--warn-text); }
.pill-ext { background: var(--danger-bg);  color: var(--danger-text); }
table { width: 100%; border-collapse: collapse; }
thead th { background: var(--bg-secondary); color: var(--text-secondary); font-weight: 500; font-size: 11px; text-transform: uppercase; letter-spacing: 0.05em; padding: 10px 12px; text-align: left; border-bottom: 1px solid var(--border-strong); }
tbody td { padding: 9px 12px; border-bottom: 0.5px solid var(--border); vertical-align: top; line-height: 1.6; }
tbody tr:hover td { background: var(--bg-secondary); }
.section-row td { background: var(--bg-tertiary); color: var(--text-secondary); font-weight: 500; font-size: 11px; text-transform: uppercase; letter-spacing: 0.05em; padding: 6px 12px; }
.script { font-weight: 500; font-family: 'SF Mono', 'Fira Code', monospace; font-size: 12px; color: var(--text-primary); }
.path { font-family: 'SF Mono', 'Fira Code', monospace; font-size: 11px; color: var(--text-secondary); }
.note { font-size: 11px; color: var(--text-secondary); margin-top: 4px; }
</style>
 
<table>
  <thead>
    <tr>
      <th style="width:22%">Script</th>
      <th style="width:38%">Paths</th>
      <th>Outputs produced</th>
    </tr>
  </thead>
  <tbody>
    <tr class="section-row"><td colspan="3">🏃 Simulation &amp; Benchmark</td></tr>
    <tr>
      <td class="script">run_benchmark_dr_pa.py</td>
      <td>
        <span class="pill pill-in">IN</span> <span class="path">../data/knowledge_base_extract.json</span><br>
        <span class="pill pill-dep">DEP</span> <span class="path">src/orchestrator.py → MedSimOrchestrator</span><br>
        <span class="pill pill-out">OUT</span> <span class="path">../results/benchmark_simulation/{model}/patient_{p}_doctor_{d}/{pathology}/transcript.json</span>
      </td>
      <td>
        Generates <strong>25 combinations</strong> of Patient×Doctor (5×5 models), one per pathology.<br>
        Each <span class="path">transcript.json</span> contains: full transcript, heuristic scores, LLM judge scores (patient + doctor).
      </td>
    </tr>
    <tr>
      <td class="script">run_benchmark.py</td>
      <td>
        <span class="pill pill-in">IN</span> <span class="path">../results/biomistral/transcript_*.json</span><br>
        <span class="pill pill-in">IN</span> <span class="path">../data/knowledge_base_extract.json</span><br>
        <span class="pill pill-dep">DEP</span> <span class="path">src/orchestrator.py → MedSimOrchestrator</span><br>
        <span class="pill pill-out">OUT</span> <span class="path">../results/benchmark/{model}/transcript_*.json</span>
      </td>
      <td>
        Evaluates existing BioMistral transcripts with each judge model.<br>
        Appends a <span class="path">benchmark_{model_key}</span> key to each output JSON file.<br>
        <span class="note">⚠️ Requires pre-generated transcripts in <span class="path">../results/biomistral/</span></span>
      </td>
    </tr>
    <tr class="section-row"><td colspan="3">📊 CSV Extraction</td></tr>
    <tr>
      <td class="script">extract_benchmark_results_dr_pa.py</td>
      <td>
        <span class="pill pill-in">IN</span> <span class="path">../results/benchmark_simulation/{model}/patient_{p}_doctor_{d}/{pathology}/transcript.json</span><br>
        <span class="pill pill-out">OUT</span> <span class="path">../results/simulation_benchmark_summary.csv</span>
      </td>
      <td>
        Recursively traverses the <span class="path">benchmark_simulation/</span> directory tree.<br>
        Produces 1 CSV with 25 rows containing: config, heuristic scores, LLM judge scores, metrics, final diagnosis.
      </td>
    </tr>
    <tr>
      <td class="script">extract_benchmark_results.py</td>
      <td>
        <span class="pill pill-in">IN</span> <span class="path">../results/benchmark/{model}/transcript_*.json</span><br>
        <span class="pill pill-out">OUT</span> <span class="path">../results/ultimate_benchmark_summary.csv</span>
      </td>
      <td>
        1 row per clinical case (pathology). Columns per model:<br>
        <span class="path">{MODEL} - Accuracy (/5) · Reasoning (/10) · Safety (/5) · Total Grade (/20) · Feedback</span><br>
        <span class="note">⚠️ Expects a <span class="path">benchmark_{model}</span> key in each JSON — depends on <span class="path">run_benchmark.py</span></span>
      </td>
    </tr>
 
    <tr class="section-row"><td colspan="3">📈 Graph Generation</td></tr>
    <tr>
      <td class="script">generate_graphs_dr_pa.py</td>
      <td>
        <span class="pill pill-in">IN</span> <span class="path">../results/simulation_benchmark_summary.csv</span><br>
        <span class="pill pill-out">OUT</span> <span class="path">graphs_simulation/1_average_grades_by_role.png</span><br>
        <span class="pill pill-out">OUT</span> <span class="path">graphs_simulation/2_heatmap_heuristic.png</span><br>
        <span class="pill pill-out">OUT</span> <span class="path">graphs_simulation/3_heatmap_llm_judge.png</span><br>
        <span class="pill pill-out">OUT</span> <span class="path">graphs_simulation/4_boxplots_by_role.png</span><br>
        <span class="pill pill-out">OUT</span> <span class="path">graphs_simulation/5_llm_judge_subscores_combined.png</span><br>
        <span class="pill pill-out">OUT</span> <span class="path">graphs_simulation/6_heuristic_vs_llm_by_model.png</span><br>
        <span class="pill pill-out">OUT</span> <span class="path">graphs_simulation/7_top_combinations.png</span>
      </td>
      <td>
        7 charts from the simulation CSV.<br>
        Charts 3, 5, and 6 are automatically skipped if no LLM judge data is present.
      </td>
    </tr>
    <tr>
      <td class="script">generate_graphs.py</td>
      <td>
        <span class="pill pill-in">IN</span> <span class="path">../results/ultimate_benchmark_summary.csv</span><br>
        <span class="pill pill-out">OUT</span> <span class="path">graphs/1_average_grades.png</span><br>
        <span class="pill pill-out">OUT</span> <span class="path">graphs/2_heatmap.png</span><br>
        <span class="pill pill-out">OUT</span> <span class="path">graphs/3_boxplots_severity.png</span>
      </td>
      <td>
        3 charts for the legacy single-judge pipeline (1 model = 1 column).<br>
        Expects columns named <span class="path">{MODEL} - Total Grade (/20)</span>.
      </td>
    </tr>
  </tbody>
</table>
---

## Model Registry
 
| Key | Model | HuggingFace Path |
|-----|-------|-----------------|
| `biomistral` | BioMistral 7B DARE | `BioMistral/BioMistral-7B-DARE` |
| `llama3` | Meta Llama-3.1 8B | `meta-llama/Llama-3.1-8B-Instruct` |
| `mistral` | Mistral 7B v0.3 | `mistralai/Mistral-7B-Instruct-v0.3` |
| `gemma2` | Google Gemma-2 9B | `google/gemma-2-9b-it` |
| `phi3` | Microsoft Phi-3 Mini | `microsoft/Phi-3-mini-4k-instruct` |

## 👨‍💻 Authors & Acknowledgments

* **Arthur Prevel**
* **Aloïs Kamber**

Developed at **UiT - The Arctic University of Norway**.  
Data sourced ethically from the **NCBI StatPearls Database**.