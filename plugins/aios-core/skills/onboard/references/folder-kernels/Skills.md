# Skills (the personalization of the skills)

> [!important] Language: these instructions are in English; the user is Hebrew-speaking. Reason and run the steps in English, but write everything the user sees in Hebrew: chat replies, reports, vault notes, tables, status lines, task text. Keep commands, file paths, field names, dates, and numbers as they are.

This is where the material that tailors the skills to you lives: notes in your voice, work preferences, and examples a skill loads at runtime. The skill itself (the SKILL.md) comes from the plugin and is not here. This folder is **yours**: whatever you write here, the skill will respect. Whatever is written here for you is written in Hebrew.

## Structure

```
Skills/{skill-name}/
  notes.md          Notes in your voice, work preferences
  strategy.md       Your overall approach to this skill
  references/
    {topic}.md      Examples, templates, deeper context
```

## Rules

- One folder per skill. The folder name matches the skill slug (kebab-case).
- `notes.md` is for short directives in your voice that the skill should respect (for example "always start from the numbers", "do not use emojis").
- `references/` is for examples and long material the skill loads on demand.
- Do not duplicate the content of the SKILL.md itself here. This folder is for your customizations, not for a copy of the skill.
- Use `[[wikilinks]]` for every project, client, or resource mentioned.
