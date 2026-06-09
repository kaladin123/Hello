Here is the extracted text from the image:
You are a senior Python / Google ADK / A2A engineer working across two repositories: cirrus-aaps_altus-external (the ingress layer) and cirrus-aaps_altus-supervisor (the Supervisor/Host Agent). I want to do two things: (1) understand how the code currently works, and (2) see what changes are needed to pass structured data from the Supervisor to the worker agent instead of a free-text prose summary.
CONTEXT - THE PROBLEM IN PLAIN WORDS
Currently the Supervisor collects multiple turns from a user conversation and assembles them into a prose sentence before calling the worker agent. For example:
 * Turn 1: "move the person to group 2"
 * Turn 2: "Mahesh"
 * Supervisor currently sends: "move the Mahesh person to group 2" (a generated sentence)
This free-text generation is where unintended rewrites sneak in (e.g. "SOP" -> "standard operating procedures" in Jira US10522665). The fix is to send structured data instead of a prose sentence, so the worker receives typed fields and the user's exact words are preserved verbatim in a separate audit field.
STEP 1 - UNDERSTAND THE CURRENT CODE (read-only, no changes yet)
 1. In cirrus-aaps_altus-external, find the entry point where the user's message is received. Trace the code path from that entry point to where the message is handed off to the Supervisor. Show me the exact file paths, function names, and the shape of the payload at each hop.
 2. In cirrus-aaps_altus-supervisor, find where the Supervisor reads incoming user messages and how it manages multi-turn context - specifically how it accumulates turns and decides it has enough information to call a worker agent.
 3. Find the exact line(s) where the Supervisor builds the message or instruction it sends to the worker agent via A2A. Show the file path, function name, and the current payload structure sent to the worker.
 4. Find the ADK agent definition for the Supervisor (LlmAgent or Agent subclass) - show the 'instruction', 'global_instruction', any callbacks, and any tools that touch the user message before the worker call.
 5. Find the ADK agent definitions for the worker agents (Data Exploration, Pricing Contract Load, MemGroup Renewal Date) - show what payload shape they expect to receive and how they parse it.
 6. Summarize the full flow in a simple numbered list: User message -> external repo processing -> Supervisor turn accumulation -> worker call payload -> worker action.
STEP 2 - PROPOSE THE STRUCTURED DATA CHANGE (show diffs, no edits yet)
Based on what you found above, show me the code changes needed to implement the following target structure sent from the Supervisor to the worker:
{
"intent": "<detected_intent_string>",
"entities": {
"<slot_name>": "<slot_value_verbatim_from_user>"
},
"original_utterances": ["<turn_1_verbatim>", "<turn_2_verbatim>"],
"conversation_id": "<id>"
}
Specifically:
 1. Show a before/after diff of the exact function in cirrus-aaps_altus-supervisor that currently builds the prose summary - replace it with a function that builds the structured dict above.
 2. Show how the Supervisor's ADK instruction or callback needs to change so it extracts slots (person name, group number, etc.) rather than generating a sentence.
 3. Show how each worker agent's entry point needs to change to read from structured fields (e.g. entities['person'], entities['target_group']) instead of parsing a prose string.
 4. Show where original_utterances should be populated - it must capture the user's exact words verbatim before any processing touches them.
 5. If there is a shared schema or contract file (dataclass, Pydantic model, TypedDict, JSON schema) between the two repos for the A2A payload, show where it lives and what the updated version should look like. If none exists, show where to add one.
STEP 3 - IMPACT CHECK (read-only)
 1. List every other file in both repos that reads or parses the current worker payload format - these will also need updating.
 2. List every test file that asserts on the current prose payload - these will need to be updated to assert on the structured fields.
 3. Flag any files that should NOT be touched.
OUTPUT FORMAT I EXPECT
 * For Step 1: a numbered walkthrough with repo/file:line references for every key point.
 * For Step 2: clear before/after code blocks for each change, labelled with the repo and file path.
 * For Step 3: a simple list grouped as "files to update" and "files to not touch."
Do not make any edits to files yet. Show me the analysis and proposed diffs first, then wait for my confirmation before changing anything.
