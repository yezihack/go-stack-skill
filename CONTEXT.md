# Go Stack Skill

A distribution context for a bundle of opinionated Go technology-selection skills. It exists so that one set of conventions can be installed into many different agent-driven coding environments.

## Language

**Skill bundle (技能包)**:
A single distributable unit that contains one or more skills.
_Avoid_: repository, plugin pack, library

**Skill (技能)**:
A self-contained convention unit that tells an agent which libraries and architecture to use in a target project.
_Avoid_: rule, prompt, capability, template

**Target project (目标项目)**:
A codebase into which a skill is installed and whose technical choices the skill governs.
_Avoid_: this project, consumer, host

## Relationships

- A **Skill bundle** contains one or more **Skills**
- A **Skill** is installed into a **Target project** and governs that project's technical choices

## Example dialogue

> **Dev:** "我装了 go-stack-skill 这个技能包，两个技能都会生效吗？"
> **Maintainer:** "默认整包安装；你也可以只要其中一个，例如只装 go-web-stack 到目标项目。"

## Flagged ambiguities

- "skill" was used both for the whole repository and for an individual unit — resolved: the repository is a **Skill bundle**, each `SKILL.md` unit is a **Skill**.
