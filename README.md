# Digital Filter
An AI agent system that filter noisy information from Internet into high-quality personal ones.

The goal is to filter information to high-quaulity products. Here is the illustration of the information pipeline:
1. Unstructured human-picked data: Personalized Information Sources (PIS), including X bookmarks, LinkedIn likes, alpharxiv, saved blog feed. The data are unstructured.
2. Structured data: Well-organized for index with initial summaries.
3. Insights and relations: Through user-AI collaboration, further enhance the indexed data with more insights and clarified relations.
4. Gap identification/literature review: Analyze and identify key gaps in areas and generate high-quality literature reviews.
5. Hypothesis generation: Generate new hypothesis or methodology that fill gaps.

Major tech methods:
* Design claude skills to automate process step in the infomation pipeline.
* Memory: Use a markdown file as notepad to record TODOs and important notes.

File structure:
- `./skills/`: Store developed skills.
- `./memory/`: Store memory files shared by all skills.
- `./data/`: All generated data. Organize by skill name.

How to use. 
* Copy your skills folder to `./.claude/skills/` and run `ln -s ./skills/<skill-name> .claude/skills/<skill-name>`. The system will automatically load all skills in the folder. Or you can add to your user skill folder `~/.claude/skills/`.
* In order to collect thru browser, run `claude --chrome`. Otherwise, the claude cannot connect to chrome.

Key skills:
1. `collect`: Incrementally retrieve information from Personalized Information Sources (PIS), including X bookmarks, alphxiv, saved blog feed and github awesome lists or repos. Example:
  - `/collect x-bookmarks`: Use `/chrome` (instead of Playwright MCP) to open `x.com/i/bookmarks` and retrieve the latest 20 posts. Note down the 
  - `/collect github`: Retrieve from `https://github.com/jyhong836?tab=stars`.
  - `/collect alphaxiv-lib`: Use `/chrome` to open https://www.alphaxiv.org/bookmarks?folder=0194fd43-1bcc-7b7e-a534-98e9dd04a943 and retrieve all records. 
  - `/collect alphaxiv-rec`: Use `/chrome` to open https://www.alphaxiv.org/?sort=Recommended and retrieve top 20 records.
2. `organize`: Process collected data from 1 to create/update structured index in a unified markdown file. Deduplicate same papers. Each posts from PIS should be categorized with metadata (author, URL, key words, insights). Organize as table format with similar collumsn as 1 but with additional column to refer to the original source file (for example, x-bookmarks.md). Each big theme is a markdown file. Each sub-topic is a section in markdown file.
  - `/organize update`: Update (or create if not exists) markdown files.
3. `assistant-read`: Help the user to explore new unread content. Reading should be recorded in the memory file. The system should be able to recommend what to read based on user's reading history and the new content. After reading, user can also update the data with new insights.
  - `/assistant-read`: Auto Select a topic that was recently added or updated.
  - `/assistant-read <topic>`: Find all posts related to the topic. Provide different options to read: Read latest, read AI recommended, read the one most related to the last reading.
  - `/assistant-read <command e.g., add insight to> <post name>: <insight>`: Update the data with new insights to the corresponding post.
4. (optional) `analyze-gap`: Identify key trend and emergent insights from the organized stream.
  - `/analyze-gap <topic: e.g., llm agent>`: Find all related post, generate a new markdown report to summarize literatures and identify gaps.

Potential extension (not included in the repo):
1. In addition to literature, tools from github should also be organized.
2. Support more sources:
   1. Email subscription (e.g., arxiv digest, newsletter, etc.)
   2. Browser history/opening tabs (e.g., chrome history, etc.)
   3. Wechat accounts.
   4. Any web pages.
3. Personalization. The system can learn from user's feedback and reading history to further personalize the recommendation.
   1. Skill/tool generation.
   2. Experience forming.
   3. Hypothesis generation.
   4. Personal tastes in picking papers and generating insights.

Tools used for building the system:
* Agent design is based on [agent-skills-for-context-engineering](https://github.com/muratcankoylan/Agent-Skills-for-Context-Engineering).
* Design principles are inspired by [clear thinking](./references/clear_thinking.md).

## Design principles.

Goal: Clear Thinking
1. "Create space for thinking" -- Filter noise from the internet and create a clean, organized, and insightful personal knowledge base, experience sets, etc.
2. "Use the space to think clearly" -- Through assistant reading, AI will help to generate questions, retrieve answers and help quickly form experiences from reading. Further, you can prompt to generate new ideas.

What we want to remove:
* Complicated operations of archiving, downloading, searching and taking note.

What we want to add:
* A clear, organized, and insightful personal knowledge base that can be referred to on demand.
* A tutor-style reading experience that helps users to focus and to quickly form insights and experiences from reading.


