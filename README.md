# ConvBondThesis

Python notebooks accompanying thesis research on **convertible bond valuation with binomial lattice models**. The project implements a Cox–Ross–Rubinstein (CRR) convertible bond model, compares alternative tree parameterizations, and explores how prices change as the number of time steps increases.

The repository also includes vanilla option benchmarks and a notebook that renders a manually populated Greeks table.

## Repository contents

| File | Purpose |
| --- | --- |
| [CRR+Alternative+ConvergenceAnalysis.ipynb](CRR%2BAlternative%2BConvergenceAnalysis.ipynb) | Main convertible bond implementation: stock, bond value and conversion probability trees; European and American conversion; comparisons across five models; convergence plots. |
| [CRR Benchmark.ipynb](CRR%20Benchmark.ipynb) | Black–Scholes–Merton calculations, European and American CRR option pricing, a call price surface, and comparisons of alternative option trees. |
| [OptionGreeksTable.ipynb](OptionGreeksTable.ipynb) | Formats predefined Delta, Gamma, Rho, Vega and Theta values into a table and saves a PNG. It does not calculate Greeks from the pricing functions. |
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

These are research notebooks with illustrative assumptions. Keep the following details in mind when interpreting or extending the experiments:

- **Volatility is hard-coded in the multi-model bond routines.** Both `LatticeConvertibleBond` and `getPrice_LatticeModel` set `sigma = 0.2` internally. Changing the notebook's global `sigma` does not change their valuation volatility. The separate `CRR_convertible_bond` function accepts `sigma` explicitly.
- **Conversion defaults differ.** `CRR_convertible_bond` defaults to European conversion, `LatticeConvertibleBond` defaults to American conversion, and `getPrice_LatticeModel` always checks immediate conversion.
- **Dividend and carry inputs are not handled uniformly.** CRR probability calculations use `r`, while Jarrow–Rudd and Haahtela incorporate `q`. Tian reads the global variable `b`, so changing `r` or `q` requires checking that `b = r - q` remains consistent.
- **Coupon timing uses a custom ceiling-based rule.** It is not a general payment-date schedule. Review its alignment with the tree grid before changing maturity or the number of steps.
- **The option benchmark needs care.** The function `calculate_option_price`, used in the section labelled European convergence, includes an early-exercise check. With the default non-dividend-paying call this distinction may not be visible, but the function should not be treated as a general European pricer.
- **Two Black–Scholes expressions differ.** The first `black_scholes` helper places `b` outside the maturity multiplier in its `d1` numerator. The later explicit calculation and `black_scholes_call_price` use the conventional `(r + sigma**2 / 2) * T` term. Review the first helper before using it as a numerical reference.
- **Greeks are not computed or validated here.** The table notebook contains fixed values, without a calculation pipeline linking them to the bond models. The repository does not implement the Taylor-based validation described in the original project abstract.
- **Input validation is limited.** Check positive maturity, volatility and step count, valid branch probabilities, and admissible up/down factors before using other parameter sets.
- **The credit treatment is simplified.** There is no explicit default/recovery process, market calibration, stochastic rate model, issuer call schedule or holder put schedule.
- **Compatibility may require minor maintenance.** The table notebook uses `DataFrame.applymap`; its saved output records a deprecation warning. Depending on the pandas version, this may need updating to `DataFrame.map`.

Saved notebook outputs document previous runs; they are not an automated correctness check. A full numerical validation is required before relying on these implementations beyond the illustrated research examples.

## License

No license file is currently included in the repository.
