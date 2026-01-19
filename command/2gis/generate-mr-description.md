# Command: generate Gitlab MR description

## Purpouse

Analyze the changes in this feature branch and generate a short, information‑dense merge request description for the reviewer.

## Guidelines

Follow the structure from `.gitlab/merge_request_templates/Default.md` with sections:

- `## Что сделано и зачем`
- `## На что обратить внимание в ревью`
- `## На что обратить внимание при тестировании`
  - in this section right at the top insert the following links to demo branches (instead of XXXX put the current branch number, it will be in format like PRO-123-feature-description. The 123 here is the branch number)

  ```md
  - [Ссылка на RU демо-стенд](https://urbi-pro-pro-XXXX.web-staging.2gis.ru)
  - [Ссылка на AE демо-стенд](https://urbi-pro-pro-XXXX.web-staging.urbi.ae)
  ```

- Use concise bullet points, avoid repeating obvious diffs, and focus on what the reviewer and QA really need to know.
- Make sure its in Russian
- Make it human friendly
- Focus on intentions and goals of the changes, not on the implementation details

Respond with **only** the final description in valid Markdown, no extra commentary or backticks.

