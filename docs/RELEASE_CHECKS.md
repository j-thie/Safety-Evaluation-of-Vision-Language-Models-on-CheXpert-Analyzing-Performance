# Release Checks

Resolve or explicitly document these items before claiming exact reproduction.

## Shared inference issues

- [ ] `PROMPT_NAME` changes only the output filename and does not select the prompt.
- [ ] Prompt text and parser label set are consistent in every script.
- [ ] The exact model/checkpoint revision is recorded.
- [ ] Peak VRAM and actual runtime are measured for every model and condition.
- [ ] Output naming is standardized; current scripts inconsistently include an underscore before `PROMPT_NAME`.

## MedGemma issues

- [ ] `MG_unknown_3.py` is verified to omit the image as intended.
- [ ] No-image scripts use a consistent generation-token limit.
- [ ] The original MedGemma launcher is replaced or corrected.

## Ministral issues

- [ ] No-image scripts stop loading/base64-encoding images that are never passed to the model.
- [ ] `mis_known_3.py` replace literal `\\n` sequences with actual newlines if that was not intentional.
- [ ] The four no-image scripts preserve source identifiers without requiring the image file to exist.
- [ ] The original Ministral launcher is replaced or corrected.
- [ ] Confirm whether `max_model_len=32000` is required and report its memory impact.

## OFD-image issues

- [ ] Set and record a random seed.
- [ ] Preserve the source CheXpert patient/image identifiers.
- [ ] Store the selected irrelevant-image mapping.
- [ ] Avoid grouping results under identifiers derived from ImageNet paths.
- [ ] Document the OFD-image dataset.
## SLURM issues

- [ ] Replace placeholder `/add/wanted/script.py`.
- [ ] Remove unsupported `--input_folder`, `--output_folder`, and `--batch_size` arguments.
- [ ] Ensure the environment name matches the committed setup instructions.
- [ ] Confirm that the one-hour wall time is sufficient for a full run.
- [ ] Add safe log output/error paths.

## Final clean-clone test

- [ ] Install both environments from committed files.
- [ ] Run one MedGemma smoke test.
- [ ] Run one Ministral smoke test.
- [ ] Run every documented condition.
- [ ] Generate all paper tables and figures.
- [ ] Confirm no private paths, credentials, patient data, or restricted assets are committed.


