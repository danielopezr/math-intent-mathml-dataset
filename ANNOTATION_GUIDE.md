# Annotation guide

Status: **agreed by the team (2026-11-15); integrals, visible products,
operators applied to an expression, vectors, matrices and mixed numbers added
on 2026-11-17; partial derivatives, ∇, norms and operator applications added
the same day with the new calculus crops.** It applies equally to the labels
of this dataset, to the deterministic annotator and the language model of the
`image-to-mathml` pipeline, and to the evaluation metrics. Where the pipeline
has a setting for a rule, it is named, in case the rule is revisited.

General principle: **annotate what changes or clarifies the reading; what is
already read correctly by default is not annotated.** An intent that repeats
the default reading does not help a screen reader user and is one more place
to make a mistake.

## 1. Implicit multiplication is annotated

Adjacent factors with no visible operator are annotated with `times`
(pipeline setting `IMAGE_TO_MATHML_IMPLICIT_TIMES`, on).

| Expression | Annotation |
|---|---|
| `2x` | `times(2, x)` |
| `2x^2y` | `times(2, power(x,2), y)` |
| `i(1+r)^t(1+r)` | `times(i, power((1+r),t), (1+r))` |
| `2f(x)` | `times(2, f(x))` |
| `a(b+c)`, `n(n+1)` | `times(a, (b+c))`: a parenthesised expression is one factor |
| `2n!` | `times(2, factorial(n))` |
| `\int 2x\,dx` | `times(2, x)`; the `dx` is not a factor |

These are not factors: the `d` of a differential (`dx`, and `dt^n` in
`d^n/dt^n`), named functions (`sin`, `sin^2`), operator names (`dim`, `ker`),
large operators and limits. Two numbers in a row are never a product (they are
digits or indices, like the "1 1" of `a_{11}`), and inside a subscript adjacent
letters are a list of indices (`a_{ij}`), not a product; a numeric coefficient
does mark one (`a_{2n}`).

## 2. Products and sums are flat

- The associative operators `times`, `plus`, `and`, `or`, `union` and
  `intersection` are written **flat, with all their arguments**:
  `times($a,$b,$c)`, not `times($a,times($b,$c))`.
- `minus`, `divided-by` and `power` take **exactly 2** arguments.
- The metrics treat the flat and the nested forms as equal.
- Pipeline setting: `IMAGE_TO_MATHML_NARY_ASSOCIATIVE` (on). It could not be
  confirmed in the W3C intent specification that an infix operator accepts
  more than two arguments (Content MathML does); if that turns out to be wrong,
  turning the setting off makes the pipeline nest them in pairs.

## 3. Function applications

- `f(x)`, `g(t)`, `f_i(x)`, `f'(x)` and operator names
  (`\operatorname{sgn}`) are applications: `$name($var)`.
- A symbol followed by a parenthesis with **a single variable** is also a
  function: `T(x)`, `Q(x)`, `y_0(t)`.
- A symbol that names a function or operator in the context (`f`, `g`, `T`,
  `D`, `P_n`, `y_n`…) is also applied to a parenthesised **expression**:
  `T(e_k)`, `D(1 + x)`, `f(x + h)` → `$name($var)`. A coefficient before a
  parenthesis is still a product: `a(b+c)`, `n(n+1)` → `times`.
- A parenthesis with commas, `T(x, y)`, is an argument list: `$name($v1, $v2)`,
  not a product and not a pair.
- Named functions use their concept: `sin x` / `sen x` → `sine($var)`,
  `ln x` → `natural-logarithm($var)`, `Γ(x)` → `gamma-function($var)`.
- Without parentheses a named function takes the whole implicit product that
  follows it: `cos kx` → `cosine($var)` with `kx` as the argument; `sin θ cos θ`
  is `times(sin θ, cos θ)`. A power of the function goes on the application:
  `cos^2 θ` → `power(cosine($var), 2)` on the group `cos^2 θ`.

## 4. Default readings are not annotated

A fraction in `mfrac`, a subscript in `msub` and a root in `msqrt`/`mroot`
**carry no intent**: their default reading is already right. These are
annotated:

- the inline fraction `a/b` → `fraction($num,$den)`;
- powers → `power($base,$exp)` (on the `msup`; never on `sin` in `sin^2 x`,
  nor on an `msubsup` without nesting the index);
- `e^x` → `exponential($exp)`;
- unusual uses of an `mfrac`, such as the binomial coefficient, and the mixed
  number `3 1/2` → `mixed-fraction($int,$frac)` on the group of both.

Also not annotated, because the symbol already says it:

- products with a visible operator, `A × B`, `2 × 3`, `a · b` (in `A × B` the
  `×` may also be a Cartesian product);
- large operators with limits: integrals, sums, products and limits
  (`∫_a^b`, `∑_{k=0}^{n}`, `lim_{x→0}`), and `sup` / `inf`;
- subscripted symbols such as `e_k` or `v_1`, and bold letters (`r`, `F`,
  `i`, `j`, `k`): the font already marks a vector and no intent is added;
- maps between sets, `X: T → S`.

Numbers are written as in the source: a fraction is always an `mfrac`, never
a `½` character.

Pipeline setting: `IMAGE_TO_MATHML_ANNOTATE_DEFAULTS` (off). The pipeline also
removes these intents if the model adds them.

## 5. Notation that depends on the topic

- `⟨u, v⟩` is always `inner-product`.
- `(u, v)` is `inner-product` **only if the topic** is about inner products,
  orthogonality or orthogonalisation (Gram-Schmidt); otherwise it is left
  unannotated. Pipeline setting:
  `IMAGE_TO_MATHML_INNER_PRODUCT_PAIRS=auto|on|off`.
- That is why the `topic` column of the metadata should be filled in.

## 6. Derivatives with primes

`y'`, `y''`, `y'''` and `y^{(n)}` are annotated as `derivative($y, _x, n)`,
with the variable as a **literal** (`_x`) because it does not appear in the
expression. This applies in calculus or differential-equation contexts
(`IMAGE_TO_MATHML_DERIVATIVE_PRIMES=auto|on|off`; the variable is set with
`IMAGE_TO_MATHML_DERIVATIVE_VARIABLE`). The prime may be written next to its
base or as a superscript: the metrics treat both forms as equal.

## 7. Other readings

- Partial derivatives are annotated on the `mfrac`:
  `partial-derivative($f, n, $x)` with the order `n` and the variables in the
  order of the denominator: `∂f/∂x` → `partial-derivative($f, 1, $x)`,
  `∂²f/∂x²` → `partial-derivative($f, 2, $x)`, `∂²f/∂y∂x` →
  `partial-derivative($f, 2, $y, $x)`. The operator `∂/∂θ` alone is not
  annotated; applied to a parenthesis, `∂/∂θ (∂f/∂x)`, the group of both is a
  partial derivative of the parenthesis. The Jacobian notation
  `∂(F, G)/∂(x, y)` is not annotated.
- `∇f` → `gradient($f)`, `∇·F` → `divergence($F)`, `∇×F` and `rot F` →
  `curl($F)`, on the group of the operator and its operand. `∇f(a)` is the
  application of `∇f` to `a`. `∇` alone (`∇ = ∂/∂x i + …`) is not annotated.
- `‖v‖` → `norm($v)` and `|x|` → `absolute-value($x)`, on the group with the
  bars; `‖y‖²` is the power of the norm.
- `D_1 f(x, y)`, `D_{ij} f(a)`, `DX(u)`: the operator and the function form
  the name of the application: `$name($v1, $v2)` with `D_1 f` as the name, not
  a product. `f'(a; y)` (directional derivative) is an application of `f'`
  with no `derivative` intent; `y'`, `Y'(x)` and `r'(t)` are derivatives with
  respect to the variable of the context (`_x`, `_t`, …).
- `Δx`, `Δu` are one factor: `c_{ij} Δx_i Δy_j` → `times(c_ij, Δx_i, Δy_j)`.
- A product written with `⋯` between factors is one `times` with all the
  factors: `(1-α)(2-α)⋯(n-α)` → `times((1-α), (2-α), (n-α))`.

- For matrices and linear maps, `A^{-1}` is `inverse($a)` and `A^T` is
  `transpose($a)` on the `msup`; numbers keep `power` (`3^{-2}`).
- A tuple or basis written with parentheses, `(1, 1 + x, 1 + x + x^2)`, and
  a list of definitions separated by commas or semicolons carry no intent.
- A table between brackets or parentheses is a matrix: `:matrix` on the
  `mtable`. With a single column it is a column vector: `vector($vec)` on the
  bracketed group, with `arg="vec"` on the `mtable`. Between bars it is a
  determinant: `determinant($mat)` on the group and `:matrix` on the `mtable`.

- `n!`, `5!` and `(n-k)!` are `factorial($var)`, on the group formed by the
  operand and the `!`. For a parenthesised operand the argument may be the
  content or the whole parenthesised group.
- An ellipsis `…` or `⋯` is an operator (`<mo>`) with `intent="ellipsis"`.
- `x → a`, `x → 0`, `x → +∞` (a lowercase letter, an arrow and a value) are
  `tends-to($var,$value)`; maps such as `f: A → B` are not.

## 8. General rules

- A single root `<mrow>` inside `<math>`, with no `display` attribute.
- Every `$ref` has exactly one descendant with `arg="ref"` in its scope.
- Only names from the W3C registry; a name that is not a concept is written as
  a literal with `_` (`_x`).
- Where the annotation goes, following W3C: delimiters (`|x|`, `‖v‖`,
  `⌊x⌋`, `⟨u,v⟩`) and `n!` on the group.

## Pending

- Verify in the W3C specification the multiple arguments of `times`.
