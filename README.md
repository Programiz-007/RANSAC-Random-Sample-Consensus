# RANSAC Line Fitting — Live Visualization

A from-scratch implementation of RANSAC (RANdom SAmple Consensus) for robust line
fitting, with no fitting/estimation library involved — only `numpy` for array
math and `matplotlib` for visualization. The point of this project is to make
the algorithm's behavior visible frame by frame, not just print a final answer.

## Algorithm

RANSAC fits a model to data that contains outliers by repeating three steps:

1. **Guess inliers** — pick a minimal random sample of points (2, for a line).
2. **Compute model** — fit a candidate model through that sample.
3. **Score the model** — test every data point against the candidate and count
   how many agree with it (the inliers).

The candidate with the highest inlier count after all iterations is kept, then
refined using a proper fit over its full inlier set.

### How many iterations are needed?

Instead of guessing an iteration count, it's computed from a standard closed-form
result:

```
        log(1 - p)
N = ------------------
    log(1 - (1-e)^s)
```

- **p** — desired probability of drawing at least one all-inlier sample (0.99 here)
- **e** — outlier ratio in the data
- **s** — minimal sample size (2 for a line)

This is implemented in `required_iterations()`. As the outlier ratio grows, the
required N grows sharply, since the odds of drawing two clean points at random
drop fast.

## Implementation notes

- **Line representation**: lines are stored in implicit form `a·x + b·y + c = 0`
  rather than `y = m·x + b`, so vertical or near-vertical lines don't break the fit.
- **Distance / scoring**: point-to-line distance is computed directly from the
  implicit form (`|a·x + b·y + c|`), no library distance function used.
- **Final refit**: once the winning inlier set is known, a total-least-squares
  refit is done by hand via the closed-form eigen-decomposition of a 2×2 scatter
  matrix — not `np.polyfit` or `np.linalg.eig` — so every step stays derivable.

## Visualization

Running the script opens a two-panel animation:

- **Left panel**: all data points, the current random sample (black X's), the
  current candidate line (dashed red), that candidate's inliers (orange), and
  the best model found so far (solid green) with its inliers (green).
- **Right panel**: inlier count per iteration (the score) plotted live, with a
  staircase line tracking the best score seen so far.

## Usage

Run the python notebook in a suitable environemnt.

Behavior adapts to the environment automatically:

- **Normal desktop Python** (interactive backend): opens a live animated window.
- **Google Colab**: renders the animation as an embedded HTML5 player
  (play/pause/scrub) directly in the output cell, since Colab has no display.
- **Other headless/non-interactive environments**: saves the animation to
  `ransac_animation.gif` instead of opening a window.

## Tuning

Key parameters, set in the `__main__` block:

- `outlier_ratio` (in `make_data`) — fraction of generated points that are outliers.
- `threshold` (in `ransac_line`) — distance under which a point counts as an inlier.
- `success_prob` (in `required_iterations`) — confidence level driving the
  iteration count formula.

Raising `outlier_ratio` toward 0.6–0.7 is a good way to see the iteration count
formula's exponential growth in action, and to watch RANSAC need more attempts
before it converges on the true line.

## Files

- `RANSAC.ipynb` — the full implementation and animation.
