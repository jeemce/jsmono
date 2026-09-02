# AGENT ENTRYPOINT

**MANDATORY BOOTSTRAP PROTOCOL:**
Before processing any user request or analyzing code, execute this exact sequence:

1. **CALL TOOL:** Read `./.agents/BRAINS.md` IMMEDIATELY.
2. **APPLY:** Enforce all directives in `BRAINS.md` as non-negotiable system rules for this session.
3. **PROCEED:** Only after reading `BRAINS.md`, begin executing the user prompt.

*Constraint: Do not skip Step 1. Do not load raw context files in `./.agents/brains/` until `BRAINS.md` logic is initialized.*
