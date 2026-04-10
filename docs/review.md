# **_Skepthical_** review: *Sparse Identification of Inviscid Fluid Dynamics from High-Dimensional Spatial-Temporal Data*

## Summary

The manuscript applies Sparse Identification of Nonlinear Dynamics (SINDy) to infer governing PDEs from a 3D fluid simulation dataset consisting of $10$ time snapshots of density and three velocity components on a $128^3$ periodic grid (Sec. 2.1). Spatial and temporal derivatives are computed using second-order central finite differences (Sec. 2.2), and a feature library is constructed from raw fields, first/second (including mixed) derivatives, and nonlinear polynomial/advection-type terms (Sec. 2.3). Sparse regression is performed via ridge regression with iterative hard thresholding across a regularization path, with BIC used for model selection (Sec. 2.4). The identified momentum equations for $v_x$, $v_y$, $v_z$ resemble inviscid Euler-like dynamics with consistent coefficients across components and a gradient-of-density term interpreted as pressure-related (Sec. 3.2). The authors further infer an effective physical time step and an implied sound speed by rescaling coefficients (Sec. 3.2–4.1). The density equation is acknowledged to be poorly identified (very low $R^2$) due to near-constant density and noisy temporal derivatives, though selected terms are argued to be qualitatively consistent with continuity (Sec. 3.1–3.2). Evaluation consists of in-sample fit metrics and a one-step forward Euler prediction between two snapshots showing low error for velocity fields (Sec. 3.3–3.4). Overall, the paper is clearly written and the recovery of Euler-like momentum structure from high-dimensional 3D data is promising, but key claims (Euler identification, time-step/sound-speed inference, and physical interpretability of coefficients) currently rest on underspecified scaling/nondimensionalization, limited validation beyond a single short-horizon case study, and insufficient robustness/sensitivity analyses.

## Strengths

- Clear, well-organized presentation of the end-to-end SINDy workflow (data $\rightarrow$ derivatives $\rightarrow$ library $\rightarrow$ sparse regression $\rightarrow$ evaluation) (Sec. 2).
- Compelling demonstration on a genuinely high-dimensional 3D dataset ($128^3$ grid, four fields) rather than small 1D/2D toy problems (Sec. 2.1).
- Recovered velocity equations exhibit physically interpretable structure close to Euler-like momentum dynamics, with near-isotropic coefficients across $x/y/z$ components (Sec. 3.2).
- The manuscript is candid about failure modes (notably the density equation) and discusses plausible causes (small $\partial\rho/\partial t$, differentiation noise) rather than overstating success (Sec. 3.1–3.2, Sec. 4.1).
- Uses multiple diagnostics ($R^2$, MSE, residual histograms, spatial error maps) and a forward prediction experiment, providing more evidence than coefficient tables alone (Sec. 3.3–3.4).
- Helpful qualitative interpretation of artifacts such as apparently canceling polynomial/second-derivative terms and why they may not be physically meaningful (Sec. 3.2).

## Major issues

1.  **The manuscript’s central physical interpretation (recovery of inviscid Euler-like dynamics and a pressure-gradient mechanism) is not rigorously validated because the data-generating PDE(s), closure (equation of state / pressure model), physical parameters (e.g., viscosity), and numerical scheme are not explicitly specified or compared term-by-term against the discovered models (Sec. 1, Sec. 2.1, Sec. 3.2, Sec. 4.1).** Without ground-truth disclosure or a quantitative comparison, it is difficult to distinguish true equation recovery from a qualitatively similar surrogate enabled by correlations in the dataset (e.g., $\nabla\rho$ acting as a proxy for $\nabla p$ under some regimes).
    
    *Recommendation:* Add a dedicated description of the simulator in Sec. 2.1 (or a new subsection): governing PDE(s) (compressible Euler/Navier–Stokes? weakly compressible model?), pressure/closure (barotropic/isothermal? incompressible Poisson pressure?), viscosity/forcing, nondimensionalization/units, discretization (finite volume/spectral/etc.), and snapshot spacing in simulation time. In Sec. 3.2, provide a side-by-side table comparing the true PDE term structure and coefficients to the discovered model (relative coefficient errors where applicable). If the ground truth is unavailable, state that clearly in Sec. 1/Sec. 2.1 and reframe claims in Sec. 3.2/Sec. 4.1 as qualitative discovery/hypothesis generation rather than “recovering Euler.”

2.  **Coefficient interpretability is currently unclear due to feature scaling/standardization and unknown nondimensionalization, yet coefficients are treated as directly physical—most critically in the inference of the physical time step $\Delta t_{\rm true}$ and sound speed from the learned advection/gradient coefficients (Sec. 2.3–2.4, Sec. 3.2, Sec. 4.1).** If columns of $\Theta$ (and possibly targets) are scaled, then reported coefficients must be unscaled before interpreting them as PDE parameters; otherwise the $\Delta t_{\rm true}$ and $c_s$ estimates can be artifacts of preprocessing. Relatedly, setting $\Delta t_{\rm true}$ so that the advective coefficient becomes $1$ risks circular reasoning unless the nondimensional form is independently justified.
    
    *Recommendation:* In Sec. 2.3–2.4, specify exactly how $\Theta$ columns were scaled ($z$-score, $L_2$ norm, max norm, etc.), whether $\partial_t X$ targets were scaled, and how coefficients were transformed back to the original units before being reported/used in Sec. 3.2. In Sec. 3.2/Sec. 4.1, explicitly list the assumptions required for the mapping “learned coefficient $\rightarrow \Delta t_{\rm true} \rightarrow$ inferred $c_s$” (unit advective coefficient in the chosen nondimensionalization; correct derivative stencil scaling; negligible viscosity; barotropic relation; $\rho \approx 1$), and—if possible—compare inferred $\Delta t_{\rm true}$ to simulation metadata. If metadata are unavailable, present $\Delta t_{\rm true}$ and $c_s$ as hypothesis-consistent effective parameters and soften wording accordingly.

3.  **Key SINDy configuration details are underspecified and robustness is not established: the library definition, ridge/threshold schedules, convergence/refit procedure, and the precise BIC computation are not fully documented; nor are there ablations/sensitivity analyses to show that the Euler-like structure is stable rather than an artifact of a particular library/penalty (Sec. 2.3–2.4, Sec. 3.2, Sec. 4.2).** This is especially important because the library is likely highly collinear in 3D, and the paper already observes “canceling” derivative combinations—suggesting identifiability/stability issues under correlated features.
    
    *Recommendation:* Expand Sec. 2.3 with a compact but explicit enumeration of library terms (categories + maximum polynomial degree + whether mixed derivatives are included) and report the library size (\#features) per target; if lengthy, include the full list in an Appendix while keeping counts in the main text. Expand Sec. 2.4 with: ridge $\lambda$ grid, hard-threshold rule (absolute/relative), number of iterations/stopping criterion, and whether coefficients are refit on the active set without ridge. Provide the exact BIC formula with definitions of $N$ and $k$ and discuss how spatial correlation affects the effective sample size. In Sec. 3.2 (or new Sec. 3.5), add robustness checks: (i) ablate second-derivative terms, (ii) ablate pure density polynomials, (iii) vary scaling/threshold/BIC penalty strength, (iv) subsample spatial points and/or hold out time slices, and report term-support stability and coefficient variability (e.g., bootstrap intervals).

4.  **Derivative estimation is a critical bottleneck given only $10$ time slices and very small temporal derivatives (especially for $\rho$), but the manuscript provides mainly qualitative discussion and no quantitative sensitivity analysis; second-order finite differences may be suboptimal for periodic domains where spectral derivatives are natural (Sec. 2.2, Sec. 3.1–3.2).** As a result, it is unclear whether failures (density) and even key recovered coefficients (velocity) are robust to more accurate/regularized differentiation.
    
    *Recommendation:* In Sec. 2.2 and Sec. 3.1–3.2, add a quantitative derivative-sensitivity study: compare current second-order central differences to (i) spectral spatial derivatives (FFT-based) given periodicity, and (ii) at least one temporal denoising/differentiation approach (e.g., Savitzky–Golay in time, smoothing splines/Tikhonov). Report how $R^2$/MSE and the main coefficients (advective and gradient terms) change. Also include an empirical estimate of derivative noise/error (e.g., forward vs. backward vs. central disagreement; magnitude of $\partial_t\rho$ relative to estimated noise floor).

5.  **The density equation identification and its interpretation as continuity-consistent are not yet technically convincing: the discovered form reported (e.g., $-0.0648\nabla\cdot \mathbf{v} + 0.0638\rho\nabla\cdot \mathbf{v}$) does not directly match $\partial_t\rho = -\mathbf{v}\cdot\nabla\rho - \rho\nabla\cdot\mathbf{v}$, and it is unclear whether $v\cdot\nabla\rho$ terms were present in the library and rejected, or absent altogether (Sec. 3.2; also affects claims in Sec. 4.1).** Given the extremely low signal-to-noise for $\partial_t\rho$, relying on $R^2$ is also problematic.
    
    *Recommendation:* In Sec. 3.2, write the full learned density equation explicitly (all selected terms and coefficients) and state explicitly whether $v_i\partial_i\rho$ terms were included in $\Theta$ and what coefficients they received along the regularization path (even if thresholded out). Add alternative evaluation metrics for the density target that are meaningful in low-variance settings (e.g., correlation, normalized residual norm, or relative error vs. a baseline). Consider learning/enforcing continuity structure directly (e.g., include $\nabla\cdot(\rho\mathbf{v})$ as a single composite feature, or impose a constraint) and report whether this improves identifiability.

6.  **Model evaluation is too limited to establish dynamical fidelity: results rely on in-sample fit and a single one-step forward Euler prediction between consecutive snapshots, which can appear strong even for imperfect models when the system evolves slowly (Sec. 2.5, Sec. 3.3–3.4).** There are no longer-horizon rollouts, stability checks, out-of-sample tests, or baseline comparisons (e.g., persistence), making it hard to assess whether the learned PDE truly captures the dynamics.
    
    *Recommendation:* Extend Sec. 3.4 to include multi-step rollouts over as many steps as the data allow (e.g., 3–5 steps), using a more stable integrator (RK2/RK4) and clearly specifying step size and any substepping. Report error growth curves (NRMSE/relative $L_2$) and include at least a persistence baseline $X(t+\Delta t) = X(t)$. Where relevant, compare distributional/structural statistics (e.g., kinetic energy, divergence norms, spectra) between predicted and true fields. If longer rollouts are infeasible, state that limitation explicitly and qualify conclusions to “one-step/short-horizon only.”

7.  **Generality/novelty is not sufficiently supported: the experimental evidence is based on a single dataset with a very short temporal window, and there is no comparison to alternative PDE-discovery approaches or to prior SINDy/PDE-FIND results in fluids to clarify what is new beyond a case study (Sec. 1–4).** 
    
    *Recommendation:* In Sec. 1 and Sec. 4.2, strengthen positioning relative to existing PDE-discovery work in fluids (representative SINDy/PDE-FIND Navier–Stokes/Euler studies, compressible/weakly compressible cases) and state clearly the novelty (e.g., 3D $128^3$ scale, limited snapshots, time-step inference). If feasible, add at least one additional scenario (different initial condition/parameter regime) or a controlled robustness test (synthetic noise injection; alternative derivative schemes) and/or a comparison to another identification baseline. Otherwise, frame the paper explicitly as a single-case feasibility/diagnostic study.

## Minor issues

1.  Important dataset context is missing or scattered: beyond periodicity, the paper does not clearly state boundary conditions, solver type, whether viscosity/forcing are present, and exact snapshot spacing; these omissions hinder interpretation of “inviscid” and the time-step inference (Sec. 2.1, Sec. 3.1).
    
    *Recommendation:* Add a concise “Data provenance” paragraph in Sec. 2.1 listing solver class, any viscosity/forcing, closure/pressure handling, and the sampling interval between the $10$ snapshots.

2.  The interpretation of the “pressure-gradient-like” term as $-c_s^2 \nabla\rho$ depends on a barotropic/isothermal assumption and on $\rho\approx 1$; alternative pressure mechanisms (e.g., incompressible Poisson pressure) could also yield a gradient term that correlates with $\nabla\rho$ in some regimes (Sec. 3.2, Sec. 4.1).
    
    *Recommendation:* Clarify in Sec. 3.2 whether the pressure model/equation of state is known. If not, temper the claim and add a diagnostic (e.g., quantify correlation between the learned gradient term and $\nabla\rho$; test inclusion of $\nabla(\rho^2)$ or other density-based gradients and report whether coefficients/fit change).

3.  The paper frequently characterizes the flow as “weakly compressible/nearly incompressible” without a quantitative criterion, which is important for interpreting the density equation and inferred $c_s$ (Sec. 3.1–3.2, Sec. 4.1).
    
    *Recommendation:* Report simple compressibility diagnostics in Sec. 3.1 or Sec. 3.2 (e.g., mean/STD of $\rho$, max $|\rho-\langle \rho\rangle|/\langle \rho\rangle$, and an estimated Mach number using typical $|\mathbf{v}|$ and inferred $c_s$).

4.  Claims about “non-physical” polynomial-density terms and canceling second-derivative combinations being negligible are largely qualitative; without magnitude comparisons, it is unclear whether they are truly irrelevant or compensating for missing physics/derivative errors (Sec. 3.2).
    
    *Recommendation:* In Sec. 3.2, compute and report RMS and max magnitudes of each selected term over space–time, normalized by the leading advective term magnitude, and (optionally) rerun identification with these terms removed/blocked to show that the main structure and predictive skill are unchanged.

5.  Forward-prediction implementation details are incomplete: the Euler step size, whether any substeps were used, and how this relates to assumed $\Delta t=1$ vs inferred $\Delta t_{\rm true}$ are unclear (Sec. 2.5, Sec. 3.4).
    
    *Recommendation:* Specify in Sec. 2.5/Sec. 3.4 the integration timestep used, number of substeps between snapshots, boundary handling during rollout, and whether coefficients were interpreted in scaled or unscaled time units during prediction.

6.  BIC model selection is described, but the effective sample size is ambiguous given strong spatial correlations in PDE data; treating all $128^3$ grid points as independent can over-penalize/under-penalize models in nonstandard ways (Sec. 2.4).
    
    *Recommendation:* In Sec. 2.4, discuss spatial correlation and how $N$ is defined for BIC. Consider reporting an additional selection criterion (e.g., held-out-time validation error) or a sensitivity analysis where $N$ is replaced by an effective $N$ based on spatial subsampling.

7.  Presentation/figure quality issues reduce interpretability: figures are often small with hard-to-read fonts; axes/colorbars frequently lack units or explicit normalization; panel identifiers are inconsistent; slice location/time index is sometimes unclear (Figures 1–6; Sec. 3.1–3.4).
    
    *Recommendation:* Increase figure size/resolution (or use vector output), add consistent panel labels (a–l) referenced in captions, specify time index and slice location (e.g., $z=L/2$) in every relevant caption, and ensure colorbars/axes state units or nondimensionalization and use consistent zero-centered scales for comparable variables.

8.  Some reported summary statistics appear inconsistent: e.g., the stated velocity-model $R^2$ range ($0.675$–$0.724$) conflicts with a listed component value $v_z=0.581$ (Sec. 3.2).
    
    *Recommendation:* Correct the range or clarify which components it refers to, and ensure all summary ranges match the reported per-component numbers.

9.  Abstract/Introduction wording can overstate generality (broad PDE discovery claims) relative to the limited evidence: single dataset, $10$ snapshots, density equation not recovered (Abstract, Sec. 1, Sec. 4.2).
    
    *Recommendation:* Revise Abstract and Sec. 1 to explicitly frame the work as a 3D case study under limited temporal sampling, and summarize key limitations upfront (derivative noise; weak density identifiability; short-horizon evaluation).

## Very minor issues

1.  Minor formatting and hierarchy inconsistencies in headings (mix of markdown-style hashes and numbered headings) slightly obscure structure (Sec. 2–3).
    
    *Recommendation:* Standardize section/subsection formatting throughout (consistent LaTeX sectioning or consistent markdown convention).

2.  Typographical/notation issues (split words, inconsistent vector/component notation such as $v_x$ vs $v_{x}$, inconsistent boldface for vectors) reduce polish (Sec. 3.2–3.4, Sec. 4.1).
    
    *Recommendation:* Proofread and standardize notation (e.g., always $\mathbf{v}$ for vectors; $v_{x}, v_{y}, v_{z}$ for components) and fix line-break artifacts.

3.  The symbol $L$ is used in figure descriptions (e.g., $z=L/2$) without a crisp definition of domain length relative to $\Delta x=1/128$ (figure captions; Sec. 2.1).
    
    *Recommendation:* Define domain lengths ($L_x, L_y, L_z$) explicitly and relate them to grid spacing and resolution.

4.  A few captions are verbose and repeat main-text narrative while omitting key quantitative context (e.g., timestep used in prediction) (Sec. 3 figures).
    
    *Recommendation:* Make captions more self-contained with essential parameters (time index, slice location, $\Delta t$ used, key metrics) and remove redundant narrative already in the text.

5.  If present in the manuscript, a nonstandard affiliation line (e.g., playful or placeholder text) is not appropriate for publication.
    
    *Recommendation:* Replace with standard institutional affiliations or omit affiliations if required by the venue template.


## Mathematical consistency audit

This section audits **symbolic/analytic** mathematical consistency (algebra, derivations, dimensional/unit checks, definition consistency).

**Maths relevance:** light

The paper contains a small set of central PDE expressions (Eqs. (1)–(4)) plus operator definitions and a key rescaling argument connecting regression coefficients to an inferred physical time step. Most mathematical content is interpretive rather than step-by-step derivation. The velocity-equation structure and the time-step rescaling are internally consistent. The main analytic weakness is the stated alignment of the discovered density model with the continuity equation: the partial term combination given does not algebraically match the claimed approximation unless additional omitted terms or approximations are provided.

### Checked items

1.  ✔ **Divergence definition** (Sec. 2.2, p.2)
    
    - **Claim:** Defines divergence as $\nabla\cdot \mathbf{v} = \partial v_x/\partial x + \partial v_y/\partial y + \partial v_z/\partial z$.
    - **Checks:** notation consistency, definition correctness
    - **Verdict:** PASS; confidence: high; impact: minor
    - **Assumptions/inputs:** $\mathbf{v} = (v_x, v_y, v_z)$ is a 3D velocity field on the grid.
    - **Notes:** Definition is standard and consistent with later usage.

2.  ✔ **Laplacian definition** (Sec. 2.2, p.2)
    
    - **Claim:** Defines Laplacian as $\nabla^2 \phi = \partial^2\phi/\partial x^2 + \partial^2\phi/\partial y^2 + \partial^2\phi/\partial z^2$.
    - **Checks:** notation consistency, definition correctness
    - **Verdict:** PASS; confidence: high; impact: minor
    - **Assumptions/inputs:** $\phi$ denotes a scalar field (e.g., $\rho$ or a velocity component).
    - **Notes:** Definition is standard and consistent with later references to second derivatives.

3.  ✔ **Usable time points after central differencing** (Sec. 2.2, p.2)
    
    - **Claim:** Using second-order central differences across $10$ time slices excludes the first and last slices, yielding $8$ usable time points for temporal derivatives.
    - **Checks:** counting/logic consistency
    - **Verdict:** PASS; confidence: high; impact: minor
    - **Assumptions/inputs:** Central difference stencil uses $(n+1)$ and $(n−1)$ time indices.
    - **Notes:** From indices $0..9$, central differences valid for $1..8$ inclusive, which is $8$ points.

4.  ✔ **$v_x$ equation matches component advection + gradient form** (Eq. (1), Sec. 3.2, p.4)
    
    - **Claim:** $\partial v_x/\partial t \approx -0.0095\, v_x\, \partial v_x/\partial x - 0.0091\, v_y\, \partial v_x/\partial y - 0.0090\, v_z\, \partial v_x/\partial z - 0.224\, \partial\rho/\partial x$.
    - **Checks:** algebraic structure, notation consistency
    - **Verdict:** PASS; confidence: high; impact: moderate
    - **Assumptions/inputs:** Advection operator for a scalar field $f$: $(\mathbf{v}\cdot\nabla)f = v_x\, \partial f/\partial x + v_y\, \partial f/\partial y + v_z\, \partial f/\partial z$.
    - **Notes:** The first three terms have the correct component-wise advection structure acting on $v_x$; the last term is a spatial gradient of $\rho$ in the $x$-direction.

5.  ✔ **$v_y$ and $v_z$ equations match component advection + gradient form** (Eqs. (2)–(3), Sec. 3.2, p.4)
    
    - **Claim:** $v_y$ and $v_z$ equations have the same advective structure with a gradient term in $y$ and $z$ respectively.
    - **Checks:** algebraic structure, cross-equation consistency
    - **Verdict:** PASS; confidence: high; impact: moderate
    - **Assumptions/inputs:** Same advection operator definition as for $v_x$. Gradients interpreted as component-wise derivatives.
    - **Notes:** Each equation uses $v_x, v_y, v_z$ multiplying the corresponding spatial derivatives of the target component ($v_y$ or $v_z$), consistent with $(\mathbf{v}\cdot\nabla)v_y$ and $(\mathbf{v}\cdot\nabla)v_z$.

6.  ✔ **Vector form identification of advective term** (Sec. 3.2 text following Eqs. (1)–(3), p.4)
    
    - **Claim:** The first three terms represent $-c\, (\mathbf{v}\cdot\nabla)\mathbf{v}$ with $c \approx 0.0094$.
    - **Checks:** algebraic identification, notation consistency
    - **Verdict:** PASS; confidence: medium; impact: moderate
    - **Assumptions/inputs:** $c$ treated as approximately equal across dimensions, despite small coefficient variations.
    - **Notes:** Component-wise, $(\mathbf{v}\cdot\nabla)\mathbf{v}$ produces exactly the listed products; taking $c\approx0.0094$ is consistent with the reported coefficients’ order and similarity.

7.  ✔ **Relation between assumed $\Delta t=1$ derivative and true physical time step** (Sec. 2.2 and Sec. 3.2 discussion of $\Delta t_{\rm true}$, pp.2 and 4–5)
    
    - **Claim:** Assuming $\Delta t=1$ in temporal differencing leads to learned coefficients scaled by the true time step, enabling inference $\Delta t_{\rm true} \approx 0.0094$ from the advection coefficient.
    - **Checks:** scaling/units logic, derivation logic
    - **Verdict:** PASS; confidence: medium; impact: moderate
    - **Assumptions/inputs:** Temporal derivative is computed by a formula proportional to $1/\Delta t$ (e.g., central difference divides by $2\Delta t$). Underlying physical PDE has advective coefficient exactly $1$ in physical time.
    - **Notes:** If the code uses $\Delta t_{\rm assumed}=1$ in $(x_{n+1}-x_{n-1})/(2\Delta t)$, the computed derivative equals $\Delta t_{\rm true}$ times the true derivative, so regression coefficients become multiplied by $\Delta t_{\rm true}$; this supports $\Delta t_{\rm true}\approx0.0094$ from $c\approx0.0094$. The exact stencil is not shown, so confidence is medium.

8.  ✔ **Rescaled momentum equation (normalization to physical time)** (Eq. (4), Sec. 3.2, p.5)
    
    - **Claim:** After rescaling by $1/0.0094$, the equation becomes $\partial \mathbf{v}/\partial t_{\rm true} = -(\mathbf{v}\cdot\nabla)\mathbf{v} - 24.1\, \nabla\rho$.
    - **Checks:** algebraic rescaling, cross-equation consistency
    - **Verdict:** PASS; confidence: high; impact: critical
    - **Assumptions/inputs:** $dt_{\rm true}$ equals the advection coefficient $c$. Pressure-gradient coefficient rescales by dividing by $dt_{\rm true}$ as well.
    - **Notes:** Dividing the RHS coefficients by $0.0094$ converts advection coefficient to $1$ and yields $\sim 0.227/0.0094 \approx 24.1$ for the gradient term, consistent with the stated value.

9.  ✔ **Barotropic pressure-gradient approximation** (Sec. 3.2 discussion, p.4)
    
    - **Claim:** With $P = c_s^2 \rho$, the Euler pressure term $-(1/\rho)\nabla P \approx -c_s^2 \nabla\rho$ for $\rho\approx1$.
    - **Checks:** algebraic manipulation, approximation logic
    - **Verdict:** PASS; confidence: high; impact: moderate
    - **Assumptions/inputs:** $\rho$ is close to $1$ everywhere (weakly compressible).
    - **Notes:** From $P=c_s^2\rho$, $\nabla P = c_s^2\nabla\rho$; then $-(1/\rho)\nabla P = -(c_s^2/\rho)\nabla\rho \approx -c_s^2\nabla\rho$ if $\rho\approx1$.

10.  ⚠ **Polynomial 'intercept correction' cancellation claim** (Sec. 3.2 paragraph after Eq. (4), p.5)
    
    - **Claim:** Terms like $112.38\rho - 56.27\rho^2 - 56.10$ are approximately zero because $\rho = 1 + \delta$ with $\delta = O(10^{-3})$.
    - **Checks:** series expansion / limiting behavior, algebraic consistency
    - **Verdict:** UNCERTAIN; confidence: medium; impact: minor
    - **Assumptions/inputs:** Coefficients shown are rounded (limited precision). $\delta$ is small and $\rho$ remains near $1$.
    - **Notes:** Expanding at $\rho=1+\delta$ gives a residual constant term $\approx (112.38-56.27-56.10)=0.01$ plus a linear term $\approx (112.38-2\cdot56.27)\delta\approx-0.16\delta$. Whether this is 'approximately zero' depends on omitted coefficient precision and on the relative scale of other terms in consistent units after any rescaling; the paper does not provide enough to verify.

11.  ✖ **Density equation vs continuity-equation interpretation** (Sec. 3.2 density discussion, p.5)
    
    - **Claim:** The identified terms ($-0.0648\nabla\cdot \mathbf{v} + 0.0638\rho\nabla\cdot \mathbf{v}$) align with $\partial_t\rho = -\nabla\cdot (\rho\mathbf{v}) \approx -\rho\nabla\cdot\mathbf{v}$.
    - **Checks:** algebraic equivalence, approximation consistency
    - **Verdict:** FAIL; confidence: high; impact: moderate
    - **Assumptions/inputs:** Only the displayed terms are the dominant selected terms for the density equation. $\rho\approx1$.
    - **Notes:** The combination equals $(0.0638\rho-0.0648)\nabla\cdot \mathbf{v} = -(0.0648-0.0638\rho)\nabla\cdot \mathbf{v}$, which is not proportional to $-\rho\nabla\cdot\mathbf{v}$ under $\rho\approx1$; instead it is approximately $-0.001\cdot(\nabla\cdot \mathbf{v})$ when $\rho\approx1$. To support the continuity interpretation, the full density model (including any $v\cdot\nabla\rho$ term and any rescaling) must be shown.

### Limitations

- Only the provided $7$ pages (parsed text and embedded equation renderings) were available; no supplementary material, appendices, or full regression output tables were provided for symbolic cross-checking.
- Key computational definitions (exact finite-difference stencil formulas for time derivatives, exact feature library column definitions, and any nondimensionalization conventions) are described verbally but not written as explicit equations, limiting verification to consistency checks rather than full derivation validation.
- Some coefficient-based arguments depend on rounding and on unreported feature scaling/normalization, which prevents definitive analytic assessment of the magnitude/cancellation of certain fitted polynomial terms.


## Numerical results audit

This section audits **numerical/empirical** consistency: reported metrics, experimental design, baseline comparisons, statistical evidence, leakage risks, and reproducibility.

$13$ of $14$ candidate numeric checks passed, covering grid resolution, array dimensionality, timepoint counting, coefficient summaries and rescalings, and basic range sanity checks for MSE. One check failed: the stated velocity $R^2$ range ($0.675$–$0.724$) conflicts with the listed $v_z$ $R^2 = 0.581$.

### Checked items

1.  ✔ **C1_grid_resolution_consistency** (p.2, Sec. 2.1 (Dataset))
    
    - **Claim:** System is on a $128^3$ periodic grid with spatial resolution $\Delta x = \Delta y = \Delta z = 1/128$.
    - **Checks:** unit-consistent numeric relationship
    - **Verdict:** PASS
    - **Notes:** Checked $dx$ vs $1/N$. Exact rational equality check.

2.  ✔ **C2_array_shape_matches_description** (p.2, Sec. 2.1 (Dataset))
    
    - **Claim:** Raw data loaded from NumPy array with dimensions $(10, 4, 128, 128, 128)$ corresponding to (time, variables, $x, y, z$).
    - **Checks:** internal consistency (dimensionality)
    - **Verdict:** PASS
    - **Notes:** Logical equality check of listed dimensions and $128^3$.

3.  ✔ **C3_usable_timepoints_from_central_difference** (p.2, Sec. 2.2 (Data preprocessing and derivative computation))
    
    - **Claim:** With $10$ time slices and second-order central differences, first and last slices excluded, resulting in $8$ usable time points.
    - **Checks:** internal consistency (count arithmetic)
    - **Verdict:** PASS
    - **Notes:** Exact integer arithmetic.

4.  ✔ **C4_advective_coeff_mean** (p.4, Eqs. (1)-(3) and paragraph below)
    
    - **Claim:** Advective coefficient is highly consistent with $c \approx 0.0094$ across all three dimensions.
    - **Checks:** cheap recomputation (summary statistic)
    - **Verdict:** PASS
    - **Notes:** Compared claimed to mean (diff_abs$=4.44444\times10^{-5}$, diff_rel$=0.00472813$); median diff_abs$=0$. Spread (max-min)$=0.0007$. Allow small spread; check mean/median closeness and max-min range.

5.  ✔ **C5_pressure_gradient_coeff_mean** (p.4, Eqs. (1)-(3) and paragraph below)
    
    - **Claim:** Pressure gradient term identified as $-k\nabla\rho$ with $k\approx0.227$.
    - **Checks:** cheap recomputation (summary statistic)
    - **Verdict:** PASS
    - **Notes:** Compared claimed to mean; also checked median. Loose tolerance given small sample and rounding.

6.  ✔ **C6_dt_true_from_advective_coeff** (p.4, paragraph below Eqs. (1)-(3))
    
    - **Claim:** True physical time step is $\Delta t_{\rm true}\approx 0.0094$ inferred because advection coefficient should be $1$ but observed $\approx0.0094$ when assuming $\Delta t=1$.
    - **Checks:** algebraic inversion
    - **Verdict:** PASS
    - **Notes:** Definitional/algebraic consistency under stated scaling. This is a definitional consistency check given the stated scaling.

7.  ✔ **C7_rescaling_factor_inverse_dt** (p.5, just above Eq. (4))
    
    - **Claim:** Rescaling factor is $(1/0.0094\approx 106.4)$.
    - **Checks:** division
    - **Verdict:** PASS
    - **Notes:** Division check. Rounding expected ($1/0.0094 = 106.3829...$).

8.  ✔ **C8_pressure_coeff_after_rescaling** (p.5, Eq. (4) derived from $k\approx0.227$ and $dt_{\rm true}\approx0.0094$)
    
    - **Claim:** After rescaling by $1/0.0094$, pressure term becomes $24.1\nabla\rho$ in Eq. (4).
    - **Checks:** multiplication
    - **Verdict:** PASS
    - **Notes:** Multiplication check. Because $k$ and $dt_{\rm true}$ are rounded ($0.227$ and $0.0094$).

9.  ✔ **C9_sound_speed_from_c2** (p.5, paragraph below Eq. (4))
    
    - **Claim:** Sound speed squared $c_s^2 \approx 24.1$ implying sound speed $c_s \approx 4.9$.
    - **Checks:** square root
    - **Verdict:** PASS
    - **Notes:** Square-root check. Rounding to one decimal place.

10.  ✔ **C10_density_polynomial_cancellation_at_rho_1** (p.5, paragraph: '$112.38\rho - 56.27\rho^2 - 56.10$')
    
    - **Claim:** Selected non-physical polynomial terms $112.38\rho - 56.27\rho^2 - 56.10$ evaluate to approximately zero given $\rho = 1 + \delta$ with $\delta \sim O(10^{-3})$.
    - **Checks:** cheap recomputation (baseline evaluation)
    - **Verdict:** PASS
    - **Notes:** Smallness check over rho in $\{1, 1\pm\delta, 1\pm2\delta\}$. Check smallness, not exact zero, due to rounding of coefficients.

11.  ✔ **C11_density_sigma_consistency_approx** (p.3 ($\sigma_\rho\approx 0.002$) and p.4 ($\sigma\approx2.18\times10^{-4}$ for $\partial\rho/\partial t$))
    
    - **Claim:** Density standard deviation $\sigma_\rho\approx0.002$, while temporal derivative of density has variance/STD $\sigma \approx 2.18\times10^{-4}$.
    - **Checks:** order-of-magnitude comparison
    - **Verdict:** PASS
    - **Notes:** Heuristic: ratio should be small; flagged if $>0.5$.

12.  ✔ **C12_forward_prediction_mse_range_velocity** (p.6 Sec. 3.4 and p.7 Conclusions)
    
    - **Claim:** Forward prediction MSE for velocity components is approximately $3.3\times10^{-5}$ to $5.3\times10^{-5}$.
    - **Checks:** range consistency
    - **Verdict:** PASS
    - **Notes:** Range ordering and magnitude sanity check.

13.  ✔ **C13_model_eval_mse_order_range** (p.5 Sec. 3.3)
    
    - **Claim:** Model MSE are on the order of $3\times10^{-5}$ to $6\times10^{-5}$.
    - **Checks:** range consistency
    - **Verdict:** PASS
    - **Notes:** Range ordering and magnitude sanity check.

14.  ✖ **C14_r2_range_statement_velocity** (p.6 Conclusions (also p.5 Sec. 3.3))
    
    - **Claim:** High $R^2$ values ($0.675$ to $0.724$) for velocity models; individually $R^2$ of $0.675$, $0.724$, $0.581$ for $v_x$, $v_y$, $v_z$.
    - **Checks:** range vs listed values
    - **Verdict:** FAIL
    - **Notes:** Exact inclusion check of all listed velocity $R^2$ values in claimed range.

### Limitations

- Only parsed text content was available; no access to underlying NumPy dataset, SINDy regression outputs, or numerical arrays used to compute statistics and errors.
- No plot digitization or pixel-based extraction was performed; any numeric claims apparently read from figures (e.g., maxima in captions) are listed as unverified.
- Several checks (e.g., physical interpretation of coefficients, conservation) are model-dependent; candidates focus on arithmetic/internal consistency rather than validating physical correctness.


## Paper Ratings

| Dimension | Score |
|-----------|:-----:|
| **Overall** | 5/10 █████░░░░░ |
| **Soundness** | 5/10 █████░░░░░ |
| **Novelty** | 6/10 ██████░░░░ |
| **Significance** | 5/10 █████░░░░░ |
| **Clarity** | 6/10 ██████░░░░ |
| **Evidence Quality** | 4/10 ████░░░░░░ |

**Justification:** The paper clearly demonstrates SINDy on a large 3D fluid dataset and recovers Euler-like momentum structure with consistent coefficients, but key physical interpretations (recovery of Euler, dt and sound-speed inference) rely on underspecified scaling/nondimensionalization and absent ground-truth simulator details. The Mathematical Audit flags a failure in the continuity interpretation of the learned density equation, and the Numerical Audit notes a reporting inconsistency in R² ranges; more broadly, evaluation is limited to in-sample fits and a single one-step rollout without robustness checks or ablations. These weaknesses in methodological rigor and evidence temper the impact and significance despite promising initial results and clear presentation. Overall, this reads as a solid feasibility case study that requires stronger validation and specification to support its central claims.
