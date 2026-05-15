# CrewAI

> Framework for orchestrating role-playing autonomous AI agents with collaborative intelligence, enabling production-ready multi-agent systems with guardrails, memory, knowledge management, and observability.

| Field | Value |
|-------|-------|
| Group | Agent Frameworks |
| Type | SDK |
| Open Source | Yes |
| GitHub | [https://github.com/crewAIInc/crewAI](https://github.com/crewAIInc/crewAI) |
| Stars | 51401 |
| Documentation | [Official Docs](https://docs.crewai.com/) |

## Overview

CrewAI is a Python framework for building multi-agent AI systems where autonomous agents collaborate through defined roles, goals, and backstories. Each agent operates as a specialized unit capable of using tools, maintaining memory, accessing knowledge bases, and producing structured outputs. Agents are organized into crews that coordinate work through configurable process patterns -- sequential, hierarchical, or hybrid -- enabling complex workflows to be decomposed into discrete, manageable tasks.

The framework provides a declarative approach to agent orchestration: agents and tasks are defined in YAML configuration files, while crew assembly and execution logic use Python decorators. CrewAI also introduces Flows, a higher-level orchestration primitive for building stateful, resumable workflows with event-driven routing between steps.

## Core Concepts

**Agents** are autonomous units defined by a role, goal, and backstory. Each agent can be assigned tools, memory capabilities, knowledge bases, and structured output schemas (via Pydantic models). Agents reason about their tasks and decide how to use their available tools to produce results.

**Tasks** represent discrete units of work with a description, expected output format, and an assigned agent. Tasks define what needs to be accomplished and capture the output for downstream consumption by other tasks or the final crew result.

**Crews** are collections of agents and tasks that work together toward a common objective. A crew defines the composition of the team, the process pattern for execution, and lifecycle hooks for pre- and post-processing.

**Processes** govern how tasks are executed within a crew. Sequential processes run tasks in order, passing outputs from one task to the next. Hierarchical processes use a manager agent to delegate tasks dynamically. Hybrid patterns combine both approaches.

**Flows** provide orchestration above the crew level. Flows use `@start`, `@listen`, and `@router` decorators to define step sequences, manage shared state across steps, support conditional branching, and enable execution persistence for resumable long-running workflows.

**Tools** are capabilities assigned to agents that extend their ability to interact with external systems. Built-in tools include web search (SerperDevTool), file operations, code execution, and more. Custom tools can be created by implementing the tool interface.

## Architecture

CrewAI follows a layered architecture:

1. **Configuration Layer** -- YAML files declare agents and tasks declaratively, separating orchestration logic from agent definitions
2. **Assembly Layer** -- Python classes use decorators (`@agent`, `@task`, `@crew`) to wire agents, tasks, and crews together programmatically
3. **Execution Layer** -- The process engine manages task scheduling, agent invocation, tool usage, memory persistence, and output collection
4. **Flow Layer** -- Higher-level orchestration composes multiple crews and steps into stateful, event-driven workflows with conditional routing

Agents interact with LLMs through a provider-agnostic interface. Tool execution follows a ReAct-style loop where agents reason about which tool to call, observe the result, and decide the next action. Memory systems (short-term, long-term, entity memory) persist across task executions within a crew run.

## Key Features and Functionality

- **Role-Based Agent Design** -- agents are defined with role, goal, and backstory for focused, contextual behavior
- **Declarative Configuration** -- YAML-based agent and task definitions with Python decorator-based crew assembly
- **Multiple Process Patterns** -- sequential, hierarchical, and hybrid execution strategies
- **Structured Outputs** -- Pydantic model integration for type-safe, validated agent outputs
- **Memory Management** -- short-term, long-term, and entity memory systems for context retention across tasks
- **Knowledge Bases** -- agents can access domain-specific knowledge sources during reasoning
- **Flow Orchestration** -- stateful, resumable workflows with event-driven routing and conditional branching
- **Guardrails** -- built-in validation and safety mechanisms for production deployments
- **Observability** -- monitoring and tracing capabilities for debugging and performance analysis
- **Lifecycle Hooks** -- `@before_kickoff` and `@after_kickoff` decorators for pre- and post-processing logic

## Use Cases

- **Research Automation** -- multi-agent teams that gather, analyze, and synthesize information from multiple sources
- **Content Generation Pipelines** -- sequential workflows where research agents feed writing agents that feed editing agents
- **Customer Support Triage** -- hierarchical crews where a manager agent delegates incoming requests to specialized agents
- **Data Processing Workflows** -- flows that coordinate extraction, transformation, validation, and loading across multiple agents
- **Code Review and Analysis** -- agents with specialized roles (security reviewer, performance analyst, style checker) collaborating on code assessment

## API Reference Summary

**Crew Class Decorators:**
- `@agent` -- registers a method as an agent factory
- `@task` -- registers a method as a task factory
- `@crew` -- registers a method as the crew assembly point
- `@before_kickoff` -- hook executed before crew starts
- `@after_kickoff` -- hook executed after crew completes

**Flow Decorators:**
- `@start` -- marks the entry point of a flow
- `@listen` -- subscribes a step to events from other steps
- `@router` -- defines conditional branching logic based on step outputs

**Key Classes:**
- `Agent` -- autonomous unit with role, goal, backstory, tools, and memory
- `Task` -- work unit with description, expected output, and agent assignment
- `Crew` -- collection of agents and tasks with process configuration
- `Flow` -- orchestration container for multi-step, stateful workflows
- `Process` -- enum defining execution pattern (sequential, hierarchical)

## Configuration and Customization

Agent configuration in `agents.yaml`:

```yaml
researcher:
  role: Senior Data Researcher
  goal: Uncover cutting-edge developments in AI and data science
  backstory: >
    You are a seasoned researcher with a knack for uncovering the latest
    developments in AI and data science.
```

Task configuration in `tasks.yaml`:

```yaml
research_task:
  description: >
    Conduct thorough research about {topic}.
    Identify key trends, breakthrough technologies, and potential impacts.
  expected_output: >
    A comprehensive report with main findings, structured as bullet points.
  agent: researcher
```

Environment variables configure LLM providers and API keys. The `crewai` CLI manages project dependencies and execution environments.

## Integration Patterns

**Tool Integration** -- agents are extended with tools that wrap external APIs, databases, file systems, or any callable functionality. Tools follow a standard interface with name, description, and execution method.

**LLM Provider Integration** -- CrewAI supports multiple LLM backends through a provider-agnostic configuration, allowing agents within the same crew to use different models.

**Enterprise Triggers** -- CrewAI Enterprise supports integration triggers from Gmail, Slack, Salesforce, Outlook, Microsoft Teams, OneDrive, and HubSpot for event-driven crew activation.

**External Agent Systems** -- crews can invoke existing CrewAI automations and Amazon Bedrock Agents as part of their workflows.

## Examples

Minimal crew definition in `crew.py`:

```python
from crewai import Agent, Crew, Process, Task
from crewai.project import CrewBase, agent, crew, task
from crewai_tools import SerperDevTool

@CrewBase
class LatestAiDevelopmentCrew:
    agents_config = "config/agents.yaml"
    tasks_config = "config/tasks.yaml"

    @agent
    def researcher(self) -> Agent:
        return Agent(
            config=self.agents_config["researcher"],
            tools=[SerperDevTool()],
            verbose=True,
        )

    @agent
    def reporting_analyst(self) -> Agent:
        return Agent(
            config=self.agents_config["reporting_analyst"],
            verbose=True,
        )

    @task
    def research_task(self) -> Task:
        return Task(config=self.tasks_config["research_task"])

    @task
    def reporting_task(self) -> Task:
        return Task(
            config=self.tasks_config["reporting_task"],
            output_file="output/report.md",
        )

    @crew
    def crew(self) -> Crew:
        return Crew(
            agents=self.agents,
            tasks=self.tasks,
            process=Process.sequential,
            verbose=True,
        )
```

Flow example with state management:

```python
from crewai.flow.flow import Flow, listen, start
from pydantic import BaseModel

class ResearchState(BaseModel):
    topic: str = ""
    findings: str = ""
    report: str = ""

class ResearchFlow(Flow[ResearchState]):
    @start()
    def gather_research(self):
        self.state.findings = "Research findings here"
        return self.state.findings

    @listen(gather_research)
    def write_report(self, findings):
        self.state.report = f"Report based on: {findings}"
        return self.state.report
```

## Limitations and Considerations

- Agent reasoning quality is bounded by the underlying LLM capabilities and prompt engineering
- Hierarchical processes depend on a manager agent that may introduce additional latency and token costs
- Complex multi-agent interactions can be difficult to debug without adequate observability tooling
- Memory systems add overhead and may not be necessary for simple, stateless workflows
- Enterprise features (triggers, RBAC, monitoring) require the hosted CrewAI platform and are not available in the open-source edition

## Changelog Highlights

CrewAI has evolved from a simple multi-agent framework to a production-oriented platform. Key milestones include the introduction of Flows for higher-level orchestration, YAML-based declarative configuration, structured output support via Pydantic, knowledge base integration, and the CrewAI Enterprise platform with managed deployment, monitoring, and enterprise integration triggers.

## Citations

- [1] [CrewAI Documentation](https://docs.crewai.com/)
- [2] [CrewAI Quickstart](https://docs.crewai.com/quickstart)

## Discovery Signals

> Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search.

### Keywords

crewai, crew, agent role, goal, backstory, task, process, sequential process, hierarchical process, manager agent, flow, role-based agents, yaml configuration, declarative agents, multi-agent collaboration, before_kickoff, after_kickoff, @agent decorator, @task decorator, @crew decorator, @listen, @router, @start, structured outputs, entity memory, knowledge bases, crewai enterprise

### Verb-Noun Tasks

- Define agents with a role, goal, and backstory in YAML
- Compose tasks with description, expected output, and assigned agent
- Assemble a crew with sequential or hierarchical process
- Add a manager agent for hierarchical task delegation
- Build a stateful resumable Flow with `@start`, `@listen`, `@router`
- Validate agent output against a Pydantic schema
- Hook into crew lifecycle with `@before_kickoff` and `@after_kickoff`
- Equip agents with SerperDevTool, file tools, and code execution
- Connect a crew to Gmail/Slack/Salesforce triggers via CrewAI Enterprise
- Wire short-term, long-term, and entity memory into agents

### User Intent Phrases

- "How do I build a multi-agent team with different roles?"
- "How do I orchestrate a researcher agent and a writer agent in sequence?"
- "How do I define agents declaratively in YAML?"
- "How do I make a manager agent delegate to specialists?"
- "How do I build a content generation pipeline with multiple agents?"
- "How do I add memory to my CrewAI agents?"
- "How do I create a resumable workflow with CrewAI Flows?"
- "How do I trigger a crew from Slack or Gmail?"
- "How do I assign different LLMs to different agents in the same crew?"
- "How do I enforce a structured output schema on an agent?"
- "How do I add guardrails to a production crew?"

### Problem Statements

- Need to decompose a task into specialized agents with distinct roles and responsibilities
- Want declarative YAML-driven agent and task definitions to avoid hand-wiring orchestration code
- Hierarchical workflows need a routing manager but introduce extra latency and token cost
- Multi-agent interactions are hard to debug without crew-level tracing
- Memory subsystems add overhead for simple stateless flows
- Enterprise integrations (triggers, RBAC, hosted monitoring) require the paid platform
- Building a stateful resumable multi-step workflow on top of plain agents is too low-level

### When to Pick This

- Pick this when you want role-based multi-agent teams (role, goal, backstory) and declarative YAML configuration of agents and tasks
- Pick this over LangChain when team composition and sequential/hierarchical task flow matter more than chain composition
- Pick this over LangGraph when you prefer declarative role-based abstractions to explicit graph topology
- Pick this over AutoGen when you want structured sequential/hierarchical processes rather than conversational message-passing patterns
- Pick this over Pydantic AI when team coordination matters more than single-agent type safety
- Pick this over smolagents when you need crew-level orchestration, lifecycle hooks, and YAML configs instead of a minimalist core
- Pick this over Semantic Kernel when role-based crews fit better than .NET-style plugin orchestration
- Pick this over ADK when you want a Python-only multi-agent framework with simple roles rather than a hierarchical agent tree tied to Google's ecosystem

### Related Terms and Aliases

- Crew (team of agents)
- Role-based agent design
- Sequential / hierarchical / hybrid process
- Manager agent
- Flow (`@start`, `@listen`, `@router`)
- ReAct loop (tool use pattern)
- Entity memory
- Knowledge base integration
- CrewAI Enterprise (hosted platform)
- Lifecycle hooks (`@before_kickoff`, `@after_kickoff`)
- Multi-agent system
- Declarative agent configuration
