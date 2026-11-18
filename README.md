# Math intent MathML dataset

Images of mathematical expressions cropped from Spanish-language textbook
pages, with hand-written **Presentation MathML** labels that carry
**`intent`** and **`arg`** annotations (the MathML intent mechanism), so a
screen reader can say what an expression means and not only how it looks.

It is the evaluation set of the `image-to-mathml` pipeline (image → LaTeX →
annotated MathML), but the labels follow a written annotation guide and can be
used on their own.

## Contents

| Area | Folder | ID prefix | Images | Labelled | Pages |
|---|---|---|---|---|---|
| Algebra | `areas/algebra` | `balg` | 220 | 220 | 29 |
| Calculus and differential equations | `areas/calculus-differential-equations` | `calc` | 220 | 220 | 22 |
| Linear algebra | `areas/linear-algebra` | `alg` | 220 | 220 | 46 |
| Discrete mathematics | `areas/discrete-math` | `dic` | 60 | 10 | 39 |
| Mathematical logic | `areas/mathematical-logic` | `log` | 67 | 0 | 32 |
| Probability | `areas/probability` | `prob` | 61 | 0 | 34 |
| **Total** | | | **838** | **670** | **202** |

The textbooks are in Spanish, so some expressions contain Spanish function
names and words (`sen` for sine, `y` for "and").

## Layout

```
areas/<area>/
  images/   one cropped expression per file: <id>.jpeg
  labels/   its annotated MathML: <id>.mml (only for labelled images)
  pages/    the full textbook pages: <prefix>-page-NN.jpeg
metadata/<area>.csv   one row per image
metadata/pages.csv    each page with its area and the original photo number
ANNOTATION_GUIDE.md   how the labels are annotated
```

The pages are photographs of the books: they have been straightened and
cropped to the paper (without the table, fingers or the facing page), and
duplicate photos were removed. Some linear-algebra pages are close-ups of part
of a page.

### Metadata

Each `metadata/<area>.csv` has one row per image, with these columns:

| Column | Meaning |
|---|---|
| `id` | Sample identifier, the file name without extension |
| `image` | Image path, relative to the area folder |
| `label` | Label path, relative to the area folder; empty if not labelled yet |
| `area` | Subject area |
| `topic` | Textbook section the expression comes from (`;` separates several); empty if unknown |
| `complexity` | `very simple`, `simple`, `medium` or `complex`; empty if not labelled |

The topic matters for some readings: a round pair `(u, v)` is annotated as an
inner product only when the topic is about inner products or orthogonality.

## Labels

Each label is one UTF-8 MathML document with a single root `<mrow>`. Intents use
concept names from the W3C intent registry; implicit multiplication, function
applications, powers, factorials, ellipses and other readings are annotated as
described in [`ANNOTATION_GUIDE.md`](ANNOTATION_GUIDE.md). In short:

```xml
<math xmlns="http://www.w3.org/1998/Math/MathML">
  <mrow intent="times($a, $b)">
    <mn arg="a">2</mn>
    <mrow arg="b" intent="factorial($n)">
      <mi arg="n">n</mi>
      <mo>!</mo>
    </mrow>
  </mrow>
</math>
```

Labels may use numeric character references (`&#x2034;`) or the characters
themselves, and mathematical italic letters (`𝛼`) or plain ones (`α`); both are
equivalent for evaluation.

## Status

- **Reviewed and approved:** these labels were checked one by one against
  their images (2026-11-18):
  - all the calculus labels, `calc001`–`calc220`;
  - the algebra labels `balg001`–`balg104`.

  `balg105`–`balg220` and the labels of the other areas have not been
  reviewed yet.
- Mathematical logic and probability have no labels yet; discrete mathematics
  has 10 of 60. Algebra, calculus and linear algebra have 220 labelled
  images each, numbered `001`–`220` without gaps.
- 326 labelled samples have no topic.

## History

The dataset started as `image-to-mathML-dataset`. On 2026-11-17 it was moved
here and reorganised; the commit history was kept. Old paths map to new ones:

| Before | Now |
|---|---|
| `chopped/<área>/<id>.jpeg` | `areas/<area>/images/<id>.jpeg` |
| `chopped/<área>/labels/<id>.txt` | `areas/<area>/labels/<id>.mml` |
| `full-pages/<área>/` | `areas/<area>/pages/` |
| `algebra`, `calculo-ed`, `algebra-lineal`, `matematicas-discretas`, `logica-matematica`, `probabilidad` | `algebra`, `calculus-differential-equations`, `linear-algebra`, `discrete-math`, `mathematical-logic`, `probability` |

Sample IDs did not change, so results computed before the move still match.

On 2026-11-18 algebra and linear algebra were reduced to 220 labelled images
each and renumbered `001`–`220`. The unlabelled crops were removed (most were
cut off, mixed with the neighbouring line or had handwriting on them), and so
were three algebra crops that were only a piece of another sample (old
`balg012`, `balg015` and `balg057`). `metadata/id-changes-2026-11-18.csv` maps
every old ID to its new one (empty if it was removed). Results computed before
that date use the old algebra and linear-algebra IDs.

The same day three linear-algebra samples that repeated another one were
replaced, keeping their IDs, by new crops from the pages: `alg017` (it
repeated `alg016`) is now ⟨f + g, s⟩ = ∫ (f(x) + g(x)) s(x) dx, `alg074` (the
E₃ already in `alg071`) is a 2 × 2 matrix with 1/√2 entries, and `alg102`
(the same matrix as `alg054`) is T(v) = ⟨v, w₁⟩w₁ + ⟨v, w₂⟩w₂.

## License

No license has been chosen yet.
