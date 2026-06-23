# Specification: Reply Pipeline

This file is the canonical authority for reply-time processing.

Canonical flow:

input_observation -> token -> vocabulary -> grammar -> coherence search -> structural verification -> decoder projection -> collapse -> evidence logs

Reply-time processing reads aggregate.current for current pressure.

logs.current is historical aggregate evidence, not the hot current surface.
