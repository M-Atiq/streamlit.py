# streamlit.py
there is generate fast api your pc
# agents.py
from typing import Callable, Any, List
from dataclasses import dataclass
import asyncio
import types

@dataclass
class ToolCallItem:
    type: str
    output: Any = None


class ItemHelpers:
    @staticmethod
    def text_message_output(item: ToolCallItem) -> str:
        return str(item.output)


class Agent:
    def __init__(self, name: str, instructions: str, tools: List[Callable]):
        self.name = name
        self.instructions = instructions
        self.tools = {tool.__name__: tool for tool in tools}

    async def run(self, input_text: str):
        # Agent follows the instruction: call `how_many_jokes`
        tool = self.tools.get("how_many_jokes")
        if tool:
            yield ToolCallItem(type="tool_call_item")
            result = tool()
            await asyncio.sleep(1)  # Simulate delay
            yield ToolCallItem(type="tool_call_output_item", output=result)

            jokes = [f"Joke #{i+1}: Why did the chicken cross the road? To get to the other side!"
                     for i in range(result)]

            for joke in jokes:
                await asyncio.sleep(0.3)
                yield ToolCallItem(type="message_output_item", output=joke)


class Runner:
    @staticmethod
    def run_streamed(agent: Agent, input: str):
        class StreamedResult:
            def __init__(self):
                self.agent = agent
                self.input = input

            async def stream_events(self):
                yield types.SimpleNamespace(type="agent_updated_stream_event", new_agent=agent)
                async for item in agent.run(self.input):
                    yield types.SimpleNamespace(type="run_item_stream_event", item=item)

        return StreamedResult()


def function_tool(func: Callable) -> Callable:
    return func  # In a real SDK this would wrap metadata, here we just pass through
streamli secode file code
import streamlit as st
import openai

st.set_page_config(page_title="ChatGPT Stream", page_icon="🤖")

# App title
st.title("🧠 GPT Chatbot (with Streaming)")

# Secure API Key input
api_key = st.text_input("Enter your OpenAI API Key:", type="password")

# Message input
user_input = st.text_area("You:", placeholder="Ask me anything...")

# Button to send
if st.button("Send"):
    if not api_key:
        st.error("Please enter your OpenAI API key.")
    elif not user_input.strip():
        st.warning("Please type a message.")
    else:
        # Setup API key
        openai.api_key = api_key

        # Displaying chatbot response
        st.markdown("**Assistant:**")
        response_area = st.empty()

        full_response = ""
        try:
            # OpenAI streaming response
            stream = openai.ChatCompletion.create(
                model="gpt-4",  # or "gpt-3.5-turbo"
                messages=[
                    {"role": "user", "content": user_input}
                ],
                stream=True
            )

            for chunk in stream:
                if "choices" in chunk:
                    delta = chunk["choices"][0]["delta"]
                    content = delta.get("content", "")
                    full_response += content
                    response_area.markdown(full_response)
        except Exception as e:
            st.error(f"Error: {e}")
