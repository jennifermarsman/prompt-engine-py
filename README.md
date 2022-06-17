# Prompt-Refresher

This package provides an easy and reusable interface to build prompts for large scale language models (LLMs). 

Prompt Engineering is a technique used to elicit intended responses out of a LLM model and can work on many strategies.

This library is built on the strategy described in this [Microsoft Prompt Engineering](https://microsoft.github.io/prompt-engineering/) article wherein you are provided a high level description, some examples, and user interactions to help the model understand what to produce and to keep track of what it already has produced. 

As new user interactions keep coming in, the old ones are cycled out based on the max_tokens set in the ModelConfig. This ensures that the context of the model does not get too big and keeps generating meaningful responses throughout its tenure.  

## Requirements
* [Python 3.7.1+](https://www.python.org/downloads/)  

## Install

```bash
git clone https://github.com/amasandMS/Prompt-Refresher.git
cd Prompt-Refresher
python -m build
pip install .\dist\prompt_refresher-ver.x.x.x-py3-none-any.whl
```

## Simple Demo

### Code
```python
from prompt_refresher.prompt_engine import PromptEngine, PromptEngineConfig
from prompt_refresher.model_config import ModelConfig
from prompt_refresher.interaction import Interaction

config = PromptEngineConfig(ModelConfig(max_tokens=50), comment_operator = "###")
description = "This code takes in natural language utterance and generates code This code takes in natural language utterance and generates code"
examples = [Interaction("Hello", "print('Hello')"), Interaction("Goodbye", "print('Goodbye')")]
flow_reset_text = "Delete the previous objects and start afresh"
interactions = [Interaction("Hi", "print('Hi')"), Interaction("Bye", "print('Bye')")]
prompt_engine = PromptEngine(config, description, examples, flow_reset_text, interactions)

## OR ##

config = PromptEngineConfig(ModelConfig(max_tokens=50), comment_operator = "###")
description = "This code takes in natural language utterance and generates code This code takes in natural language utterance and generates code"
prompt_engine = PromptEngine(config, description)

prompt_engine.add_example(Interaction("Hello", "print('Hello')"))
prompt_engine.add_example(Interaction("Goodbye", "print('Goodbye')"))
prompt_engine.add_interaction(Interaction("Hi", "print('Hi')"))
prompt_engine.add_interaction(Interaction("Bye", "print('Bye')"))

print (prompt_engine.build_context())
```

### Output
```
### max_tokens: 50

### This code takes in natural language utterance and generates code This code takes in natural language utterance and generates code

### Hi
### Bye

## Hello
print('Hello')
## Goodbye
print('Goodbye')
## Hi
print('Hi')
## Bye
print('Bye')
```

More examples can be found in the [Examples](https://github.com/amasandMS/Prompt-Refresher/tree/main/examples) folder.

## Prompt Engineering and Context Files

To generate meaningful output output out of a large scale language model, you need to provide it with an equally descriptive prompt to coax the intended behaviour. 

A good way to achieve that is by providing helpful examples to the LLM in the form of query-answer interaction pairs. 

This is an example from the [Codex-CLI](https://github.com/microsoft/Codex-CLI) library following the above principle
```powershell
# what's the weather in New York?
(Invoke-WebRequest -uri "wttr.in/NewYork").Content

# make a git ignore with node modules and src in it
"node_modules
src" | Out-File .gitignore

# open it in notepad
notepad .gitignore
```

## Available Functions

These are the prebuilt functions that are provided by the prompt_refresher library

| Command | Parameters | Description |
|--|--|--|
| `build_context` | None | Constructs and return the context with parameters provided to the Prompt Engine |
| `build_prompt` | Prompt: str | Uses the context constructed by the build context function and generates a prompt to query  |
| `truncate_prompt` | max_tokens: int |Truncates the prompt to the max_tokens limit|
| `add_interaction` | interaction: Interaction(natural_language: str, code: str) | Adds the given natural language - code interaction to the interactionss |
| `remove_first_interaction` | None | Removes the first/most historical interaction added to the interactionss |
| `remove_last_interaction` | None | Removes the last interaction added to the interactionss |

## Adding more info to your contexts

This project comes pre-loaded with a single technique for making contexts. That doesn't mean its the best technique or the only one you should use.

Beyond these, you can build your own contexts to coax other behaviors out of the model. To change the prompt making behaviour or adding more info to the context you can inherit the PromptEngine class and change the behaviour for making the prompts. For example, 

If you want to change the behaviour for adding examples to the model:
```python
class PromptEngineOverloaded(PromptEngine):
    def _insert_examples(self):
            """
            Inserts the examples into the context
            """
            if (self.examples != []):
                for example in self.examples:
                    self.context += self.config.start_sequence + "This is an example: " + example.natural_language + self.config.stop_sequence
                    self.context += self.config.newline_operator
                    self.context += example.code + self.config.newline_operator
.
.
.

pr = PromptEngineOverloaded(...)
pr.build_context()

```
The Prompt Engine will now use the updated logic of the insert examples logic for building the context.

For in-depth look into all the functions available, please have a look at the [PromptEngine](https://github.com/amasandMS/Prompt-Refresher/blob/main/src/prompt_refresher/prompt_engine.py) class. 
