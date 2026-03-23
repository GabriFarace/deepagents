# `widgets/__init__.py`

## High-Level Purpose

This is the package initializer for the `widgets` subpackage. It contains only a docstring and no re-exports, following the convention of importing directly from submodules.

## Usage

Import widget classes directly from their submodules:

```python
from deepagents_cli.widgets.chat_input import ChatInput
from deepagents_cli.widgets.messages import AssistantMessage
from deepagents_cli.widgets.approval import ApprovalMenu
from deepagents_cli.widgets.status import StatusBar
from deepagents_cli.widgets.welcome import WelcomeBanner
from deepagents_cli.widgets.loading import LoadingWidget
from deepagents_cli.widgets.message_store import MessageStore, MessageData
from deepagents_cli.widgets.model_selector import ModelSelectorScreen
from deepagents_cli.widgets.thread_selector import ThreadSelectorScreen
from deepagents_cli.widgets.theme_selector import ThemeSelectorScreen
from deepagents_cli.widgets.mcp_viewer import MCPViewerScreen
from deepagents_cli.widgets.ask_user import AskUserMenu
from deepagents_cli.widgets.diff import compose_diff_lines
from deepagents_cli.widgets.autocomplete import MultiCompletionManager
from deepagents_cli.widgets.history import HistoryManager
```
