# quantlib-takt

QuantLib 1.43 for Python with a faster Monte Carlo core. The Python package is the official QuantLib
wheel, unchanged (`import QuantLib`); only the C++ library inside it is replaced by a TAKT build of
QuantLib 1.43. Every result is identical to the official wheel, bit for bit, on the same machine.

This free package accelerates **European options priced by Monte Carlo** on the Black–Scholes process
family. Everything else in QuantLib runs the original code and returns the original result. The
commercial version adds the other engines and much more (see "Full version" below).

Not affiliated with or endorsed by the QuantLib project or its contributors.

## Install

```sh
pip uninstall -y QuantLib        # both packages provide `import QuantLib`
pip install quantlib-takt
```

Linux x86-64, glibc 2.28 or newer (the same platforms as the official wheel), Python 3.9 to 3.13.
`import QuantLib` and the API are unchanged.

## What is faster

European-option Monte Carlo on the Black–Scholes process family: `MCEuropeanEngine` with pseudo-random
or Sobol paths, flat or curve term structures, and therefore delta/gamma/vega/rho by bumping. A book of
European options prices several times faster through the unchanged Python API; the longer the run, the
larger the share that is accelerated.

Measured end to end through the Python wheel (whole process timed: interpreter start, `import QuantLib`,
building the market and pricing), against the official QuantLib 1.43 wheel from PyPI, `takt-harness`,
9 interleaved rounds, output identical in every run:

| Workload | AMD Zen 3 | AMD Zen 4 |
|---|---:|---:|
| Book of 20 European options, non-flat curves | **5.9×** | **5.3×** |
| Book of 20 European options, flat curves | **5.5×** | **5.1×** |
| European Greeks by bumping (delta, gamma, vega, rho) | **18×** | **19×** |
| European, Sobol with Brownian bridge | **4.6×** | **4.8×** |

Inside a running interpreter (the market built once, many pricings) the same European workloads are
**7.6–8.4×** (Zen 3) and **7.9–9.0×** (Zen 4). Full measurements, both C++ and Python, are in the
showcase: https://taktcycles.com/showcase/quantlib-mc/README.html?src=github

Workloads the free package does **not** accelerate (Asian, American, barrier, digital, basket, Heston,
local volatility, …) run the original QuantLib code at the original speed, with the original result.

## The same output, bit for bit

Every number this package computes matches the official QuantLib wheel on the same machine. We check
867 printed C++ results and 270 Python results across curve shapes, day counters, calendars, path
counts, seeds, pseudo-random and Sobol, antithetic and control variates, and Greeks, plus QuantLib's
own C++ test suite (1,339 cases) and the QuantLib-SWIG Python test suite (211 cases) — all identical to
the official library.

QuantLib's results depend on the system maths library, so the official wheel itself prints slightly
different last bits on different glibc versions. quantlib-takt reproduces the **official wheel's** output
on the **same system**, bit for bit.

Check it on your own data. Price the same instruments in two virtual environments, one with `QuantLib`
and one with `quantlib-takt`, printing full precision, and compare:

```python
# verify.py — run in each venv, then `diff`
import QuantLib as ql
today = ql.Date(15, ql.May, 2025); ql.Settings.instance().evaluationDate = today
spot = ql.SimpleQuote(100.0)
r = ql.FlatForward(today, 0.03, ql.Actual365Fixed())
q = ql.FlatForward(today, 0.01, ql.Actual365Fixed())
vol = ql.BlackConstantVol(today, ql.TARGET(), 0.25, ql.Actual365Fixed())
proc = ql.BlackScholesMertonProcess(ql.QuoteHandle(spot),
        ql.YieldTermStructureHandle(q), ql.YieldTermStructureHandle(r),
        ql.BlackVolTermStructureHandle(vol))
opt = ql.VanillaOption(ql.PlainVanillaPayoff(ql.Option.Call, 105.0),
        ql.EuropeanExercise(today + ql.Period(1, ql.Years)))
opt.setPricingEngine(ql.MCEuropeanEngine(proc, "pseudorandom",
        timeSteps=252, requiredSamples=50000, seed=42))
print("%.17g %s" % (opt.NPV(), float.hex(opt.NPV())))
```

```sh
python -m venv a && a/bin/pip install -q QuantLib==1.43      && a/bin/python verify.py > a.txt
python -m venv c && c/bin/pip install -q quantlib-takt        && c/bin/python verify.py > c.txt
diff a.txt c.txt && echo "identical"
```

## Full version

The commercial licence covers everything the free package does not:

- **All Monte Carlo engines**, accelerated the same way with the same bit-identical output: Asian
  (arithmetic and geometric), American (Longstaff–Schwartz), barrier, digital, basket / multi-asset,
  Heston and Bates, and local-volatility surfaces.
- **The C++ library** (`.so` and `.a`), a drop-in for `libQuantLib` with the same ABI, for programs that
  use QuantLib from C++.
- **The free-threaded wheel** (cp314t) for Python 3.14t.
- **Builds for other platforms and QuantLib versions** on request.
- **Support** and **source escrow**: the modified source in an encrypted escrow box, released to you on
  the conditions of our Terms of Service.

Measured factors for the full version (C++, over the faster of two baselines): European 9–24×, Asian
8.8×, basket 7×, barrier 5.7×, American 4.2×, Heston 2.7×, local volatility 2.2×. See the showcase.

Prices: **Team**, $399 a month billed yearly ($4,788): all engines, the C++ library and the free-threaded
wheel on one platform, with every new QuantLib release rebuilt and checked bit for bit again.
**Enterprise**, from $1,500 a month billed yearly: other platforms and QuantLib versions, an SLA, source
escrow and acceptance on your own hardware.

New measurements of other libraries, one a week: https://t.me/taktcycles

Get a licence, a quote, or a free speed check of your own code:
https://taktcycles.com/?src=pypi-quantlib#start

## Licence

The Python bindings are QuantLib-SWIG (BSD 3-clause). The library is a modified build of QuantLib 1.43
(BSD 3-clause); the source of the modified parts is not published. Both licences are included (`LICENSE`,
`LICENSE-QuantLib-SWIG`, `NOTICE`). This package is not affiliated with the QuantLib project.

Questions, a result that differs, a build you need, a function you want faster:
https://github.com/abx-takt/quantlib-takt/issues
