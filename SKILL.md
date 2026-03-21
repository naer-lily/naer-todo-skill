---
name: naer-todo-skill
description: Manage todo tasks through a CLI interface. Use when the user needs to work with todo tasks.
---

# naer-todo-skill: Todo Task Manager

A Python CLI tool for managing todo tasks, designed for AI assistants to integrate task management capabilities. The tool provides a consistent interface while using a backend service for task storage and synchronization.

## Quick Start

### Prerequisites
- Python 3.8+
- Environment variables (for backend configuration (dida365 is used, see <https://developer.dida365.com/>)):
  ```bash
  export DIDA_TOKEN="your_backend_api_token"
  export DIDA_PROJECT_ID="your_project_id"
  ```

### Basic Commands
```bash
# List today's pending tasks (today, overdue, and unscheduled)
python scripts/main.py list-today

# List all pending tasks (all time periods)
python scripts/main.py list-all

# Create a new task
python scripts/main.py create --title "Buy groceries" --priority MEDIUM

# Complete a task
python scripts/main.py complete <task_id>

# Delete a task
python scripts/main.py delete <task_id>
```

## Core Functionality

### Task Listing
List tasks with intelligent filtering (no parameters needed):
```bash
# 列出当前未完成任务（今日以及过期的任务）
python scripts/main.py list-today

# 列出全部未完成任务（所有时间的以及没有安排时间的任务）
python scripts/main.py list-all

# 兼容命令：等同于list-today
python scripts/main.py list
```

**智能筛选特性:**
- `list-today`: 自动筛选今日任务、过期任务、以及没有安排日期的任务
- `list-all`: 列出所有未完成任务，不进行日期筛选
- `list`: 作为兼容命令，效果等同于`list-today`

**无需手动参数**: 所有筛选逻辑由脚本内部智能处理，AI助理无需传入任何筛选参数

### Task Creation
Create new tasks with detailed attributes:
```bash
python scripts/main.py create --title "Task title" [OPTIONS]
```

**Important:** Before creating tasks, check existing categories with `python scripts/main.py categories` to identify appropriate category IDs. **Note:** This skill cannot create new categories - categories must be created manually by the user in the backend interface.

**Required:**
- `--title`: Task title (required)

**Optional:**
- `--description`: Task description
- `--date`: Task date in `YYYY-MM-DD` format
- `--priority`: Task priority (`NONE`, `LOW`, `MEDIUM`, `HIGH`)
- `--category-id`: Category ID (use IDs from `categories` command)

### Task Operations
- **Get task details**: `python scripts/main.py get <task_id>`
- **Complete task**: `python scripts/main.py complete <task_id>`
- **Delete task**: `python scripts/main.py delete <task_id>`
- **List categories**: `python scripts/main.py categories` (view only - cannot create categories)

## Category Management Notes

**Important Limitations:**
- **Categories cannot be created through this skill** - categories must be created manually by the user in the backend interface
- **Categories can only be listed** - use `python scripts/main.py categories` to view existing categories and their IDs
- **Category IDs are required** - when creating tasks with categories, you must use the exact category ID from the categories list

**Workflow for category usage:**
1. User creates categories manually in the backend interface
2. Assistant checks available categories: `python scripts/main.py categories`
3. Assistant uses appropriate category IDs when creating tasks
4. No category creation, deletion, or modification is possible through this skill

## DAILY_TASK.md Integration for Recurring Tasks

The skill integrates with the `DAILY_TASK.md` file for managing recurring tasks. **This is not an automated script** - instead, the AI assistant should read this file every morning at 8:00 AM (via scheduled task) and manually create the appropriate tasks for the day.

### Implementation Approach
1. **Scheduled Assistant Wake-up**: Configure the AI assistant to wake up daily at 8:00 AM
2. **File Reading**: The assistant reads `DAILY_TASK.md` to understand recurring task patterns
3. **Manual Task Creation**: Based on current date and file content, the assistant manually creates tasks for the day using the skill's CLI
4. **Human-like Interpretation**: The assistant interprets the file's content (which can be in any format) and makes intelligent decisions about which tasks to create

### Key Principles
- **No fixed parsing scripts**: The assistant reads and interprets the file like a human would
- **Flexible format**: `DAILY_TASK.md` can be in any text format that the assistant can understand
- **Intelligent decision-making**: The assistant considers the current date, day of week, and other factors when deciding which tasks to create
- **Manual execution**: All task creation is done manually by the assistant using the CLI commands

## Configuration

### Environment Variables Setup
The skill requires two environment variables for backend integration:

1. **DIDA_TOKEN**: Backend API access token
   - Obtain from your backend service interface

2. **DIDA_PROJECT_ID**: Project identifier
   - Find in your backend service

### Dependencies
Install required Python packages:
```bash
pip install requests pydantic
```

## Output Formats

### Task List Output
```
找到 X 个任务:
================================================================================
ID | 标题 | 状态 | 优先级 | 日期 | 分类 | 描述
--------------------------------------------------------------------------------
task_id | Task Title | NORMAL/COMPLETED | 无/低/中/高 | YYYY-MM-DD | Category | Description summary
```

### Task Detail Output
```
任务ID: task_id
标题: Task Title
描述: Task description
状态: 未完成/已完成
完成时间: YYYY-MM-DD HH:MM:SS (only for completed tasks)
优先级: 无/低/中/高
日期: YYYY-MM-DD (or "未安排")
分类: Category Name (or "未分组")
```

### Success Messages
- **Creation**: `创建成功: task_id`
- **Completion**: `完成成功: task_id`
- **Deletion**: `删除成功: task_id`

## AI Assistant Response Style

When using this skill, AI assistants should follow these response guidelines:

### Constructive and Positive Responses
- After task completion or deletion, mention remaining tasks to maintain positivity
- Keep responses concise yet encouraging
- Example responses:
  - "任务已完成！当前还有[X]个待办事项，继续加油！这些待办事项是：XXX"
  - "已添加新任务。现在共有[X]个任务需要完成。这些任务是：XXX"
  - "已删除该任务。剩余[X]个任务需要处理。这些任务是：XXX"

### Minimal Output by Default
- Do not output full task lists unless explicitly requested
- Only show task details when user asks for "show/list/print my todos"
- For destructive actions without specific ID, list matching tasks first, then ask for clarification

### Error Handling Responses
- Provide clear, helpful error messages
- Suggest solutions for common issues (missing env vars, network errors)
- Maintain positive tone even when errors occur

## Usage Examples

### Example 1: List Today's Tasks
```bash
# List tasks for today (today, overdue, and unscheduled)
python scripts/main.py list-today
```

### Example 2: Create Task with Full Details
```bash
python scripts/main.py create --title "Project Review" \
  --description "Review project milestones and deliverables" \
  --date 2026-03-22 \
  --priority HIGH \
  --category-id 69be56b7ebb75f00000003fc
```

### Example 3: Complete Multiple Tasks
```bash
# First list tasks to get IDs
python scripts/main.py list-today  # or list-all for all pending tasks
# Then complete specific tasks
python scripts/main.py complete 69be63fde4b067bb55e38430
python scripts/main.py complete 69be63fde4b067bb55e38431
```

### Example 4: View Task Details
```bash
# Get detailed information about a specific task
python scripts/main.py get 69be63fde4b067bb55e38430

# List all categories
python scripts/main.py categories
```

## Important Notes

### Timezone Handling
- Backend API uses UTC time for storage
- Dates for China timezone (UTC+8) appear with time offset in API responses
- When creating tasks with dates, consider timezone differences

### Task State Management
- Cannot create tasks directly as completed
- Must create task first, then call `complete` command
- No direct task update API - delete and recreate instead

### Category Limitations
- **Categories cannot be created** through this interface
- **Categories can only be viewed** using the `categories` command
- **Category management** must be done manually in the backend interface

### Error Handling
- Missing environment variables cause immediate exit with error message
- API errors display descriptive messages
- Exit codes: `0` for success, `1` for general errors

## Best Practices for AI Usage

1. **Check First**: Always use `list` command before creating new tasks to avoid duplicates
2. **Check Categories Before Operations**: Before creating tasks, always check existing categories with `python scripts/main.py categories` to understand available groupings. **Remember:** Categories cannot be created through this skill.
3. **Clear Titles**: Use descriptive, specific titles for easy identification
4. **Regular Cleanup**: Periodically complete finished tasks and delete unnecessary ones
5. **Category Usage**: Use `categories` command to see available groupings and their IDs
6. **Error Recovery**: If API fails, check environment variables and network connection
7. **DAILY_TASK Integration**: For recurring tasks, read the DAILY_TASK.md file daily at 8:00 AM and manually create appropriate tasks based on current date and file content. The file can be in any format - interpret it intelligently like a human would.
8. **Positive Engagement**: Maintain constructive, encouraging tone in all responses

## File Structure
```
naer-todo-skill/
├── SKILL.md              # This file
└── scripts/
    ├── main.py           # Main CLI interface
    └── TaskService.py    # Backend API client library
```

## Skill Scripts

### main.py
Primary CLI interface with argument parsing and command dispatch. Handles:
- Command-line argument parsing
- Environment variable validation
- Backend API method calls
- Output formatting for AI consumption

### TaskService.py
Backend API client library providing:
- Type-safe Python interfaces for backend API
- Task and category data models
- HTTP request handling with error management
- Date/time conversion utilities

