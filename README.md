# GenAI Code Review <!-- omit in toc -->

- [1. Setup](#1-setup)
  - [1.1. Prerequisites](#11-prerequisites)
    - [1.1.1. Step 1: Create a Secret for your OpenAI API Key](#111-step-1-create-a-secret-for-your-openai-api-key)
    - [1.1.2. Step 2: Adjust Permissions](#112-step-2-adjust-permissions)
    - [1.1.3. Step 3: Create a new Github Actions workflow](#113-step-3-create-a-new-github-actions-workflow)
  - [1.2. Configuration Parameters](#12-configuration-parameters)
  - [1.3. `openai_engine`](#13-openai_engine)
    - [1.3.1. `openai_temperature`](#131-openai_temperature)
    - [1.3.2. `openai_max_tokens`](#132-openai_max_tokens)
    - [1.3.3. `mode`](#133-mode)
    - [1.3.4. `language`](#134-language)
    - [1.3.5. `custom_prompt`](#135-custom_prompt)
- [2. How it works](#2-how-it-works)
  - [2.1. files](#21-files)
  - [2.2. patch](#22-patch)
- [3. Custom Prompt](#3-custom-prompt)
  - [3.1. Overview](#31-overview)
  - [3.2. How to Use](#32-how-to-use)
  - [3.3. Potential](#33-potential)
  - [3.4. Implementation in Code](#34-implementation-in-code)
- [4. Security and Privacity](#4-security-and-privacity)
- [5. Built With](#5-built-with)
  - [5.1. Authors](#51-authors)
  - [5.2. Contributors](#52-contributors)
- [6. License](#6-license)

This project aims to automate code review using the GPT language model. It
integrates with Github Actions and, upon receiving a Pull Request, automatically
submits each code change to GPT for review.

## 1. Setup

The following steps will guide you in setting up the code review automation with GPT.

### 1.1. Prerequisites

Before you begin, you need to have the following:

- An OpenAI API Key. You will need a personal API key from OpenAI which you can
  get here: <https://openai.com/api/>. To get an OpenAI API key, you can sign up
  for an account on the OpenAI website <https://openai.com/signup/>. Once you have
  signed up, you can create a new API key from your account settings.
- A Github account and a Github repository where you want to use the code review
  automation.

#### 1.1.1. Step 1: Create a Secret for your OpenAI API Key

Create a secret for your OpenAI API Key in your Github repository or
organization with the name `OPENAI_API_KEY`. This secret will be used to
authenticate with the OpenAI API.

You can do this by going to your repository/organization's settings, navigate to
secrets and create a new secret with the name `OPENAI_API_KEY` and paste your
OpenAI API key as the value.

#### 1.1.2. Step 2: Adjust Permissions

Then you need to set up your project's permissions so that the Github Actions can write comments on Pull Requests. You can read more about this here: [automatic-token-authentication](https://docs.github.com/en/actions/security-guides/automatic-token-authentication#modifying-the-permissions-for-the-github_token)

#### 1.1.3. Step 3: Create a new Github Actions workflow

In your repository in `.github/workflows/chatgpt-review.yaml`. A sample workflow is given below:

```yaml
on:
  pull_request:
    types: [opened, synchronize]

permissions:
  issues: write

jobs:
  code_review_job:
    runs-on: ubuntu-latest
    name: ChatGPT Code Review
    steps:
      - name: GenAI Code Review
        uses: dlidstrom/genai-code-review@v3
        with:
          openai_api_key: ${{ secrets.OPENAI_API_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
          github_pr_id: ${{ github.event.number }}
          openai_model: "gpt-4o" # optional
          openai_temperature: 0.5 # optional
          openai_max_tokens: 2048 # optional
          mode: files # files or patch
          language: en # optional, default is 'en'
          custom_prompt: "" # optional
```

In the above workflow, the `pull_request` event triggers the workflow whenever a
pull request is opened or synchronized. The workflow runs on the ubuntu-latest
runner and uses the `dlidstrom/chatgpt-github-actions@v3` action.

The `OPENAI_API_KEY` is passed from the secrets context, and the `GITHUB_TOKEN` is
also passed from the secrets context. The `github_pr_id` is passed from the
`github.event.number` context. The other three input parameters, `openai_engine`,
`openai_temperature`, and `openai_max_tokens`, are optional and have default values.

Note also that you need to allow the workflow permission to comment the PR (`issues`).
This is done using these lines:

```yaml
permissions:
  issues: write
```

See [the documentation](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/controlling-permissions-for-github_token) for an explanation.

### 1.2. Configuration Parameters

### 1.3. `openai_engine`

- **Description**: The OpenAI model to use for generating responses.
- **Default**: `"gpt-3.5-turbo"`
- **Options**: Models like `gpt-4o`, `gpt-4-turbo`, etc.

#### 1.3.1. `openai_temperature`

- **Description**: Controls the creativity of the AI's responses. Higher values
  make the output more random, while lower values make it more focused and
  deterministic.
- **Default**: `0.5`
- **Range**: `0.0` to `1.0`

#### 1.3.2. `openai_max_tokens`

- **Description**: The maximum number of tokens to generate in the completion.
- **Default**: `2048`
- **Range**: Up to the model's maximum context length.

#### 1.3.3. `mode`

- **Description**: Determines the method of analysis for the pull request.
- **Options**:
  - `files`: Analyzes the files changed in the last commit.
  - `patch`: Analyzes the patch content.

#### 1.3.4. `language`

- **Description**: The language in which the review comments will be written.
- **Default**: `en` (English)
- **Options**: Any valid language code, e.g., `pt-br` for Brazilian Portuguese.

#### 1.3.5. `custom_prompt`

- **Description**: Custom instructions for the AI to follow when generating the review.
- **Default**: `""` (empty)
- **Usage**: Provide specific guidelines or focus areas for the AI's code review.

## 2. How it works

### 2.1. files

This action is triggered when a pull request is opened or updated. The action
authenticates with the OpenAI API using the provided API key, and with the
Github API using the provided token. It then selects the repository using the
provided repository name, and the pull request ID. For each commit in the pull
request, it gets the modified files, gets the file name and content, sends the
code to ChatGPT for an explanation, and adds a comment to the pull request with
ChatGPT's response.

### 2.2. patch

Every PR has a file called patch which is where the difference between 2 files,
the original and the one that was changed, is, this strategy consists of reading
this file and asking the AI to summarize the changes made to it.

Comments will appear like this:

![genaicodereview](img/genai_code_review.png "GenAI Code Review")

## 3. Custom Prompt

### 3.1. Overview

The `custom_prompt` parameter allows users to tailor the AI's review to specific
needs. By providing custom instructions, users can focus the review on
particular aspects or request additional information. This flexibility enhances
the usefulness of the AI-generated review comments.

### 3.2. How to Use

To use a custom prompt, simply provide a string with your instructions. For
example, to ask the AI to rate the code on a scale of 1 to 10, set the
`custom_prompt` parameter as follows:

```yaml
custom_prompt: "Give a rating from 1 to 10 for this code:"
```

### 3.3. Potential

Using a custom prompt can direct the AI to focus on specific areas, such as:

- Code quality and readability
- Security vulnerabilities
- Performance optimizations
- Adherence to coding standards
- Specific concerns or questions about the code

### 3.4. Implementation in Code

The custom_prompt is integrated into the review generation as shown:

```python
if custom_prompt:
      logging.info(f"Using custom prompt: {custom_prompt}")
      return f"{custom_prompt}\n### Code\n```{content}```\n\nWrite this code review in the following {language}:\n\n"
  return (f"Please review the following code for clarity, efficiency, and adherence to best practices. "
          f"Identify any ar...
```

This feature allows you to harness the power of AI in a way that best suits your specific code review requirements.

## 4. Security and Privacity

When sending code to the ChatGPT language model, it is important to consider the
security and privacy of the code because user data may be collected and used to
train and improve the model, so it's important to have proper caution and
privacy policies in place.. OpenAI takes security seriously and implements
measures to protect customer data, such as encryption of data in transit and at
rest, and implementing regular security audits and penetration testing. However,
it is still recommended to use appropriate precautions when sending sensitive or
confidential code, such as removing any sensitive information or obscuring it
before sending it to the model. Additionally, it is a good practice to use a
unique API key for each project and to keep the API key secret, for example by
storing it in a Github secret. This way, if the API key is ever compromised, it
can be easily revoked, limiting the potential impact on the user's projects.

## 5. Built With

- [OpenAI](https://openai.com/) - The AI platform used
- [Github Actions](https://github.com/features/actions) - Automation platform

### 5.1. Authors

- **CiroLini** - [cirolini](https://github.com/cirolini)

### 5.2. Contributors

- **Glauber Borges** - [glauberborges](https://github.com/glauberborges)
- **Daniel Lidström** - [dlidstrom](https://github.com/dlidstrom)

## 6. License

This project is licensed under the MIT License - see the LICENSE file for details.
