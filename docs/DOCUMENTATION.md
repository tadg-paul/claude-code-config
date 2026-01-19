# DOCUMENTATION STANDARDS

## Process
- Before devising any solution, make sure that you have read and digested the documentation for this project which may have changed since the last time you looked.
- When changing any code, update the relevant documentation after (and not before).

## Structure
- Each project should have README.md or README.org (but not both)
- Documentation should be in .md or .org format
- All other documentation should reside in ./docs/
- README should contain:
  - A high-level overview of the project
  - A Quickstart section
  - Document any project dependencies
  - A list of the important files in the project and their purpose
  - A brief overview of each doc file and a jumping off point to it
- ./docs/ should contain
  - vision: A document outlining the overall vision and goals of the project.
  - architecture: A document describing the high-level architecture of the project, includes coding standards and practices.
  - testing: A document detailing the testing strategy and procedures.
  - implementation_plan (if relevant)
  - a separate doc for each significant feature or area
  - changelog
- New projects use the MIT license with "Copyright ©[year] Taḋg Paul" unless otherwise specified.
- Do not disclose in any documentation, code or commit commments any indication that Claude Code or an AI agent was involved in the project

## Editing
- When donig any kind of find/replace, be precise in your regex matching. Simple find/replace without at least word boundaries are sloppy and bound to produce unintended consequences.

## Language
- Use en-gb-oed, that's British English *Oxford Spelling*. Your internal definition of OED spelling is wrong. To be clear, according to Wikipedia:
> Oxford spelling (also Oxford English Dictionary spelling, Oxford style, or Oxford English spelling) is a spelling standard, named after its use by the Oxford University Press, that prescribes the use of British spelling in combination with the suffix -ize in words like realize and organization instead of -ise endings.
> Oxford spelling is used by many UK-based academic journals (for example, Nature) and many international organizations (for example, the United Nations and its agencies).[1][2][3] It is common for academic, formal, and technical writing for an international readership. In digital documents, Oxford spelling may be indicated by the IETF language tag en-GB-oxendict (or historically by en-GB-oed).[4] 

## Problems
- Stop and alert me if there are any major inconsistencies in the documentation that impact the task at hand
- Warn me if there are any other inconsistencies in the documentation that do not impact the task at hand
