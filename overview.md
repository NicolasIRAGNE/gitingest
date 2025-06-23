# Gitingest V2

## Current state and problems

Gitingest is a tool that takes a git repository as input and outputs a LLM-friendly text file that can be used as context for further processing.

It is currently functional for that use case, however, some similar tools (such as [gitdiagram](https://gitdiagram.com/) or [gitcontainer](https://gitcontainer.com/)) are being developed and, while the core logic is the same (ingest git repository -> output something), they are completely new implementations.

## Goals

All these tools fundamentally boil down to a simple logic:

- Ingest a git repository
- Select the files that are relevant to the task at hand (ie. the context)
- Pass the built context to a LLM along with a system prompt

The goal of this project is to create a tool that can be used to build these tools in a more convenient way, so that the developer can focus on the logic of the tool and not the implementation details.

I view it a "context builder", as coined (afaik) by Andrej Karpathy in his [keynote at AI Startup School](https://x.com/ycombinator/status/1935496106957488566).

![image](https://github.com/user-attachments/assets/595e8486-93c4-4685-8357-70ec2cae89c0)

## Features

### URL resolution

One of the main features of the tool is the ability to be able to seamlessly ingest a directory from multiple sources:

- local (eg. `$HOME/repos/repo1`, or with URI `file:///home/user/repos/repo1`)
- remote, raw (eg. `https://example.com/source.tar.gz`)
- remote, common git providers (eg. `https://github.com/user/repo.git`). In this case, we also need to be able to target a specific branch or commit.
- ssh (eg. `git@github.com:user/repo.git`). Limited authentication support at the beginning (implicit ssh auth)

### File attributes and summarization

The tool should be able to provide basic stats and information about the files (such as size, MIME type, etc.), along with an AI-generated summary of the file (see #features-caching) that can be generated on demand.

This is a key feature, as it allows the user (or LLM agent) to quickly understand the context of the repository, and decide whether it is relevant to the task at hand, without having to pass the entire repository to the LLM, thus reducing token usage.

### Caching

Generating summaries for each file can be a very expensive operation, and can be counter-productive in case of isolated tasks.

By hosting a SaaS service, we can leverage the cache of all the users to provide a fast and efficient service.

### File selection

The tool should be able to easily filter the files, based on arbitrary criteria. We also need to handle exclusion patterns (.gitignore syntax).

An advanced feature would be to have an AI-powered file selector, that can be used to select the files based on a natural language description of the task. An example would be passing the tree to an agent, which would then output a list of files with a level of detail that it deems relevant (eg. get the entire file content, get only the summary, only attributes, etc.).

## Examples

### GitReadme

Let's say we want to build a tool that can be used to generate a README.md file for every directory in a repository.

```python
import gitingest

EXCLUDE_PATTERNS = [
    "**/node_modules/**",
    "**/dist/**",
    "**/build/**",
    "**/target/**",
    "**/vendor/**",
]

repo = gitingest.GitRepository("https://github.com/user/repo.git", exclude_patterns=EXCLUDE_PATTERNS, use_repo_gitignore=True)

tree = repo.get_tree()

# sort it by depth
tree.sort(key=lambda x: x.depth).filter(lambda x: x.type == gitingest.TreeType.DIRECTORY)

# iterate on all directories, starting for the deepest level so that we can use it in parent directories
for directory in reversed(tree):
    # get the directory content
    directory_content = directory.get_children()
    output = gitingest.call_llm(
        user_prompt=f"Generate a README.md file for the directory {directory.path}",
        pass_tree=True,
        context=directory_content,
    )
    # Add file to the tree
    directory.create_file(
        path=f"{directory.path}/README.md",
        content=output,
    )

```
