# Zacke

Math SDK, game logic, simulation, Python development. Collaborates with Max on game design and direction.

## Day-to-Day
- `game_config.py` + `gamestate.py` for each new game
- Dev sim (`int(1e4)`) to verify logic quickly
- Production sim (`int(1e6)`) once logic is confirmed
- Validate RTP + hit rate against `docs/maths-guide.md`
- Handoff: `publish_files/`, `config.ts`, event configs

## Handoff Deliverables
- `library/publish_files/` — books, LUTs, index.json
- `library/configs/config_fe_<game_id>.json` — frontend game config
- `library/configs/event_config_base.json` — base game event structure
- `library/configs/event_config_bonus.json` — bonus event structure (if applicable)

## Notes
- No formal role title — collaborates with Max
- Math correctness and RTP targeting are primary focus
- See `docs/math-sdk-notes.md` and `docs/maths-guide.md` for standards
- See `docs/setup-backend.md` for environment setup
