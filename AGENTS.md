# AGENTS.md

This file provides guidance to AI coding assistants when working with code in this repository.

## Project Overview

`bedrock-pr-reviewer` is an AI-based code reviewer and summarizer for GitHub pull requests using Amazon Bedrock's Anthropic Claude models. It is designed to be used as a GitHub Action to automatically review PRs, provide line-by-line code change suggestions, generate summaries, and support interactive conversations with the bot. This is a modified version of CodeRabbitAI's ai-pr-reviewer adapted to use Amazon Bedrock instead of direct API access.

The application is built with TypeScript and Node.js, compiled to a GitHub Action package. It uses AWS SDK for Bedrock Runtime, GitHub Actions Core and Octokit for GitHub integration, and tiktoken for token counting.

## Development Commands

### Testing

- `npm test` - Run all tests using Jest
- `npm run test -- <test-file>` - Run specific test file

### Code Quality

- `npm run lint` - Run ESLint on all TypeScript files in src/
- `npm run format` - Format all TypeScript files using Prettier
- `npm run format-check` - Check formatting without making changes

### Building & Packaging

- `npm run build` - Compile TypeScript and copy tiktoken WASM file to dist/
- `npm run package` - Build distributable package using ncc
- `npm run all` - Run build, format, lint, package, and test in sequence
- `npm run act` - Build, package, and test locally using act tool

## Architecture Overview

### Backend Structure

- **Runtime**: Node.js 20+ GitHub Action
- **AI Service**: Amazon Bedrock Runtime API with Claude models
- **GitHub Integration**: Octokit with retry and throttling plugins
- **Token Management**: tiktoken library for counting tokens in prompts
- **Concurrency**: p-limit and p-retry for controlled parallel API calls

### Key Directories

- `/src/` - TypeScript source files for the GitHub Action
- `/dist/` - Compiled distributable package (created by ncc build)
- `/lib/` - Compiled TypeScript output (intermediate)
- `/.github/workflows/` - GitHub Actions workflows for CI/CD
- `/docs/` - Documentation including CloudFormation templates for IAM setup

### Service Architecture

The application follows a modular architecture with clear separation of concerns:

- **main.ts**: Entry point that orchestrates the review process based on GitHub event type
- **bot.ts**: Handles communication with Bedrock API, manages conversation context
- **review.ts**: Core review logic for analyzing PRs and generating reviews
- **review-comment.ts**: Handles interactive conversations when users reply to bot comments
- **commenter.ts**: Manages posting and updating comments on GitHub PRs
- **octokit.ts**: Configures GitHub API client with retry and throttling
- **options.ts**: Parses and validates action inputs and configuration
- **prompts.ts**: Manages system prompts and review instructions
- **tokenizer.ts**: Token counting and management for API limits
- **limits.ts**: Defines token limits for different Claude models
- **permission.ts**: Checks user permissions for bot interactions

The bot uses two model instances:
- Light model (Claude 3 Haiku by default) for summaries and simple tasks
- Heavy model (Claude 3 Sonnet by default) for detailed code reviews

## Code Standards

### TypeScript Conventions

- Use strict TypeScript mode with noImplicitAny enabled
- Use ESNext target and module for modern JavaScript features
- Follow ESLint configuration with GitHub plugin recommendations
- Use Prettier for consistent code formatting
- Error handling: Catch exceptions and use @actions/core warning/setFailed functions
- Documentation: Use JSDoc comments for public APIs and complex logic
- Async/await for all asynchronous operations
- ESM module syntax (import/export)

### Branch & PR Naming

- Branch pattern: `feat/AUPA-<ticket-number>-<brief-description>` (based on current branch)
- Feature branches should include ticket reference when applicable
- Main branch: `main`

### Testing

- Testing framework: Jest with ts-jest for TypeScript support
- Test files: `**/*.test.ts` (excluded from compilation)
- Test configuration: jest.config.json
- Run tests with: `npm test`
- Tests should cover core review logic, API interactions, and edge cases
- Use @jest/globals for test utilities

## Key Configuration Files

- `action.yml` - GitHub Action definition with inputs, outputs, and metadata
- `tsconfig.json` - TypeScript compiler configuration
- `package.json` - Node.js dependencies and build scripts
- `.eslintrc.json` - ESLint rules and parser configuration
- `jest.config.json` - Jest test runner configuration

## Database & External Dependencies

### Required Services

- Amazon Bedrock Runtime API (AWS credentials via OIDC or IAM role)
- GitHub API (GITHUB_TOKEN provided by Actions environment)
- AWS IAM role with permissions for Bedrock model invocation

### Key External Integrations

- **Amazon Bedrock** for Claude model inference
- **GitHub REST API** for PR data, comments, and file diffs via Octokit
- **tiktoken** for accurate token counting compatible with Claude models

## Common Workflows

### Adding Support for a New Claude Model

1. Add model ID to `src/limits.ts` with appropriate token limits
2. Update default model in `action.yml` if needed (bedrock_light_model or bedrock_heavy_model)
3. Test with sample PR to verify token limits and response quality
4. Update documentation if model becomes the new default

### Modifying Review Prompts

1. Locate relevant prompt in `action.yml` (system_message, summarize, review_file_diff, or summarize_release_notes)
2. Edit the default value or note that users can override via workflow configuration
3. Test changes with sample PRs covering different scenarios
4. Consider backward compatibility for existing users

### Testing Locally with act

1. Create `.secrets` file with GITHUB_TOKEN and AWS credentials
2. Run `npm run build && npm run package`
3. Execute `./bin/act pull_request_target --secret-file .secrets`
4. Review logs for any errors or unexpected behavior

### Releasing a New Version

1. Run full build and test suite: `npm run all`
2. Commit changes and push to repository
3. Create and push a new version tag
4. Users reference the action via tag or branch in their workflows
