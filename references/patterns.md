# Teaching Style Patterns

This file stores reusable patterns for class structure and artifact design.

## Pattern 1: 教科書伴走・小刻み再現型

### Use when

- the class follows a textbook or tutorial with concrete操作 steps
- learners should touch the tool immediately after each explanation
- mixed experience levels make full self-pacing risky
- the instructor wants low-prep, repeatable class operation
- the class should feel hands-on rather than lecture-heavy

### Avoid when

- the class goal is deep discussion rather than tool use
- the material has no concrete step sequence to follow
- learners already have enough background to work autonomously for most of the session

### Core idea

Do not treat the class as `reading a textbook together`.
Treat it as `the instructor showing one small step, then learners reproducing it immediately`.

### Basic loop

1. Show one small step in `3-8 minutes`.
2. Have learners reproduce it in `10-20 minutes`.
3. Confirm an observable result.
4. Move to the next small step.

### Observable checkpoints

Use results that can be judged from the outside:

- `プロジェクトが起動する`
- `カメラ操作ができる`
- `ギミックが動く`
- `条件で反応が変わる`
- `1回プレイできる`

Avoid checkpoints that depend on hidden internal state.

### 90-minute structure

- Split one period into `2-3` small loops.
- Keep the required work finishable by most learners in `60-75` minutes.
- Use the remaining time for same-skill extensions, not unrelated extra work.

### Multi-period structure

For a day with multiple consecutive periods:

- change the _kind_ of work every 90 minutes
- front half: textbook reproduction and small modifications
- back half: summary task, application, playtest, or project integration

This prevents the day from feeling like one long repeated drill.

### Prep load

Prepare only the stable parts:

- the textbook range for that day
- the quiz
- the end-of-period or end-of-day summary task

Do not prewrite every possible extension task for every loop.
Keep only extension _types_ in mind and improvise the concrete prompt during class.

### Extension types

When a learner finishes early, extend within the same skill:

- change one condition
- reproduce the same step with a different setting
- modify one parameter and compare the result
- apply the same mechanism to the learner's own project

Avoid switching to a different skill area just to fill time.

### Implications for artifacts

#### Syllabus

- write stable time blocks and visible outcomes
- write the broad lesson shape
- do not write loop-level ad-lib extensions

#### Lesson plan

- write the loop timing more explicitly
- note what the instructor demonstrates
- note what counts as the checkpoint

#### Slides and handouts

- support the current loop only
- do not front-load long concept dumps
- place visuals close to the step they support

#### Quizzes and summary tasks

- keep them aligned with the same day's loops
- test what learners actually touched, not content they only heard about
