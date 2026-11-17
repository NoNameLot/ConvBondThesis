# ConvBondThesis

Python notebooks accompanying thesis research on **convertible bond valuation with binomial lattice models**. The project implements a Cox–Ross–Rubinstein (CRR) convertible bond model, compares alternative tree parameterizations, and explores how prices change as the number of time steps increases.

**This repository contains a selection of the code developed for the thesis, rather than the complete research codebase.** Some calculations and validation procedures discussed in the thesis are outside the published notebooks. Their absence here should not be interpreted as their absence from the thesis work.

The published material also includes vanilla option benchmarks and a notebook that renders a Greeks table from predefined values. The code used to derive those values is not included in this repository.

## Repository contents

| File | Purpose |
| --- | --- |
| [CRR+Alternative+ConvergenceAnalysis.ipynb](CRR%2BAlternative%2BConvergenceAnalysis.ipynb) | Main convertible bond implementation: stock, bond value and conversion probability trees; European and American conversion; comparisons across five models; convergence plots. |
| [CRR Benchmark.ipynb](CRR%20Benchmark.ipynb) | Black–Scholes–Merton calculations, European and American CRR option pricing, a call price surface, and comparisons of alternative option trees. |
| [OptionGreeksTable.ipynb](OptionGreeksTable.ipynb) | Formats predefined Delta, Gamma, Rho, Vega and Theta values into a table and saves a PNG. The underlying Greeks calculation code is not included in the published selection. |
| [Greeks_Table.png](Greeks_Table.png) | Committed Greeks table image. |
| [.vscode/launch.json](.vscode/launch.json) | Optional VS Code configuration for debugging the current Python file. |

There is no packaged Python module, command-line interface, dependency lockfile or automated test suite. Inputs and experiments are defined directly in notebook cells.

## Models and valuation approach

The notebooks implement these model identifiers:

| Identifier | Implementation |
| --- | --- |
| `CRR` | Reciprocal up/down factors based on volatility and a risk-neutral probability. |
| `CRR_Drift` | The CRR factors with a drift adjustment in the probability calculation. With the default `drift=0`, it is identical to CRR. |
| `Jarrow-Rudd` | Drift-adjusted up/down factors with equal branch probabilities. |
| `Tian` | Up/down factors based on exponential volatility moments and a corresponding branch probability. |
| `Haahtela` | Up/down factors based on exponential growth and a volatility-dependent spread. |

The convertible bond calculation builds a recombining stock tree and works backwards from the terminal payoff:

```text
terminal bond value = max(conversion_ratio × stock price, face value + coupon)
```

At earlier nodes, the continuation value includes the coupon logic and discounted future bond values. The discount rate on each branch blends `r` and `r + k` using the next node's conversion probability. With American conversion, the node value is the greater of continuation value and immediate conversion value. With European conversion, conversion is considered only at maturity.

The conversion probability tree is propagated from terminal conversion decisions and updated when immediate conversion is selected. It is a model quantity, not an empirically calibrated conversion forecast.

## Getting started

The notebook metadata records **Python 3.12.2**. Use Python 3.12 as a starting point; other Python and dependency versions have not been established by this repository.

Clone the repository and create a virtual environment:

```bash
git clone https://github.com/NoNameLot/ConvBondThesis.git
cd ConvBondThesis
python -m venv .venv
```

Activate it on macOS/Linux:

```bash
source .venv/bin/activate
```

Or on Windows PowerShell:

```powershell
.\.venv\Scripts\Activate.ps1
```

Install the notebook environment and imported third-party libraries:

```bash
python -m pip install jupyterlab ipykernel numpy scipy pandas matplotlib networkx
python -m jupyter lab
```

Open a notebook and select the virtual environment's Python kernel. The notebooks use in-code example parameters and do not require an external dataset or API credentials.

Dependencies are unpinned, so this setup does not reproduce an exact historical environment.

## Suggested workflow

1. Open **CRR Benchmark.ipynb** to inspect the vanilla option examples and tree comparisons.
2. Open **CRR+Alternative+ConvergenceAnalysis.ipynb** for convertible bond valuation and convergence experiments.
3. Open **OptionGreeksTable.ipynb** to reproduce the formatted table from its predefined values.

Run each notebook **from top to bottom in a fresh kernel**. Cells reuse and overwrite global variables, so running selected cells out of order can change results.

### Convertible bond example

After running the imports and the cell defining `calculate_tree_parameters` and `LatticeConvertibleBond` in the main notebook:

```python
stock_tree, bond_tree, conversion_tree = LatticeConvertibleBond(
    S0=75,
    F=100,
    T=5,
    r=0.07,
    k=0.03,
    coupon=6,
    conversion_ratio=1,
    N=5,
    model_type="CRR",
    AmeEur="a",
    q=0,
    drift=0,
)

print(f"Convertible bond value: {bond_tree[0, 0]:.4f}")
```

Use `AmeEur="a"` for American conversion or `AmeEur="e"` for European conversion. The returned arrays contain the stock prices, bond values and conversion probabilities; the value at inception is `bond_tree[0, 0]`.

### Main example parameters

| Parameter | Default example | Meaning |
| --- | --- | --- |
| `S0` | 75 | Initial stock price |
| `F` | 100 | Bond face value |
| `X` | 100 | Conversion price used to derive `F / X` |
| `T` | 5 | Maturity in years |
| `r` | 0.07 | Continuously compounded risk-free rate |
| `k` | 0.03 | Credit spread used in discounting |
| `q` | 0 | Dividend yield input; treatment varies by model |
| `sigma` | 0.20 | Volatility; fixed internally in the multi-model bond functions |
| `coupon` | 6 | Annual coupon amount, in the same monetary units as `F` |
| `conversion_ratio` | 1 | Shares received per bond; derived as `F / X` |
| `N` | 5 | Number of tree steps for the illustrated trees |

Rates and volatility are expressed as decimals. The implementation uses `dt = T / N`.

The main convergence experiments evaluate **3 through 119 steps**. The vanilla option comparisons evaluate **3 through 299 steps**; the upper bound of Python's `range` is excluded.

## Outputs

The notebooks display:

- Stock price, convertible bond value and conversion probability trees.
- Convertible bond convergence curves for all five models and a separate CRR plot.
- Vanilla option prices, a Black–Scholes call price surface and option tree convergence curves.
- A formatted Greeks table.

Running `OptionGreeksTable.ipynb` writes `greeks_table_aligned_decimals.png` to the current working directory. This filename differs from the committed `Greeks_Table.png`.

![Committed Greeks table](Greeks_Table.png)

## Implementation notes and limitations

This is a partial code release and does not provide everything needed to reproduce all thesis results. Independently validate any adaptation of the published implementations before using it beyond the illustrated research examples.

## License

No license file is currently included in the repository.
