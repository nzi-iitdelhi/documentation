# Edit this handbook

This handbook is meant to change as the work changes. If it was wrong or unclear when you
needed it, fixing it counts as part of your work.

## The quick way

Click the pencil icon at the top right of any page. GitHub opens the page in its editor; make
your change, then choose "Commit changes" and propose a pull request. You do not need to clone
anything or install any tools, and this is the way we expect most edits to happen.

## Working locally

For bigger changes, you can run the site on your own machine:

```bash
git clone git@github.com:nzi-iitdelhi/documentation.git
cd documentation
pip install -r requirements.txt
zensical serve        # http://127.0.0.1:8000/documentation/, reloads as you edit
```

Pages are plain Markdown files under `docs/`. The menu comes from the `nav:` block in
`mkdocs.yml`, so a new page only appears once you add it there. When a change is merged into
`main`, GitHub Actions publishes the site automatically.

## What belongs here

This handbook is for how the team works: roles and responsibilities, checklists, workflows
that span both repositories, the design principles and the reasons behind them, and
onboarding. How a particular module works, API and function documentation, design notes for a
single component, and changelogs all belong in the code repository, next to the code they
describe.

## How we write

Write the way you would explain something to a new colleague: in full sentences, starting with
what the reader is trying to do. A few other things help:

- When you state a rule, say what goes wrong if it is ignored. People skip rules they do not
  understand.
- Link to information instead of copying it. Copies drift apart, and one of them ends up
  wrong.
- Make sure commands can be copied and pasted, and that they work today. Try them before you
  commit.
- Use checklists only for things people actually tick off, such as a pull request or a
  release. Everything else reads better as prose.
- If you are not sure about something, mark it with a `TODO` note rather than guessing.

## Keeping it accurate

If you change a workflow this handbook describes, update the page in the same pull request as
the code. When you onboard someone, ask them to make their first pull request against the
handbook. Maintainers review the handbook as part of their monthly health check.

If someone needed a long explanation from a colleague to get going, that is a sign of a gap
here. Fixing the handbook is better than repeating the explanation for the next person.

## Reporting a problem

If something is wrong and you are not sure what the right answer is,
[open an issue](https://github.com/nzi-iitdelhi/documentation/issues). Reporting a gap is much
better than leaving it.
