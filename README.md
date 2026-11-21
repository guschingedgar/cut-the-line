# Cut the Line

Real-time protection against phone scams and AI-cloned voices: it listens to calls from unknown numbers, scores the risk live, and steps in during the call.



https://github.com/user-attachments/assets/a01cb39e-28bd-4399-ac97-c6d8b8e0bd31



## Why we built it

Phone scams aimed at older people follow the same script: a grandchild in trouble, a fake bank advisor, a fake police officer, always pushing for an urgent and secret payment. A few seconds of public audio are now enough to clone a voice, and caller ID can be spoofed. Spam filters act before the call and bank checks act after the payment. Nothing protects the conversation itself.

We built Cut the Line in 24 hours at **ESSECKATHON** (AI × cybersecurity hackathon, October 2026), Team 8. Team project with Louis Bettelli, Victor Henz and Oscar Moulet. The idea and the features came out of all four of us brainstorming together. Then we split the work: Louis and Victor built the product, and Oscar and I handled the business side. **My part was the pitch and the English deck:** the problem framing, the business model and presenting to the jury.

## What it does

- **Routes every call.** The protected person keeps their number and forwards it once to our server. Contacts pass through untouched and are never listened to. Unknown callers first hear "This call is being analyzed by a fraud protection service".
- **Scores the risk live** from four signals: Claude on Amazon Bedrock reading the transcript (45), weighted keywords (20), a synthetic-voice detector (20) and number reputation (15).
- **Responds in steps, never hangs up:** a silent warning at 40, a spoken warning to the callee plus a text to the real relative at 65, the caller put on hold at 85.
- **Checks identity out of band.** A 4-digit code is texted to the number the contact book holds for the claimed relative. A cloned voice or a spoofed number can't read it back.
- **Runs in a phone app** (installable web app) with recents, contacts, keypad and voicemail, plus a live dashboard for the demo.

## How it works

Twilio carries the call and streams the audio of unknown callers to a FastAPI server. Amazon Transcribe turns it into text, and each signal emits events that a scoring engine combines into one score. The code follows a hexagonal layout, so the scoring and intervention logic can be tested without a phone or AWS.

- **Circuit breakers** stop a strong signal from being averaged away: an unusual payment plus a claimed identity that doesn't match the number forces the score to 85 or above.
- **The voice signal is capped at 64.** Deepfake detectors generalise poorly to 8 kHz phone audio, so a voice alone can never trigger an intervention.
- **Privacy by design:** the caller is told before any audio is processed, no audio or transcript is stored, and call history keeps metadata only for 30 days.

**Stack:** Python 3.12, FastAPI, Twilio Voice (Media Streams, Conference, Voice JS SDK), Amazon Transcribe, Claude on Amazon Bedrock, wav2vec2, SQLite.

## Results

On 22 scripted calls (11 scams, 11 legitimate, in French and English), no legitimate call raised an alert. With an offline stand-in for Claude, 3 of 11 scams reach the alert level, and the grandparent scam reaches intervention 15 seconds into the call. Reports are in [`artifacts/evaluation/`](artifacts/evaluation/).

## The pitch

I built the deck and presented it to the jury: [`docs/pitch-deck.pdf`](docs/pitch-deck.pdf). It frames the problem as "most defense happens before or after the call, ours happens during it", then sets out the business case:

- **Model:** one paid monthly plan per protected line, every feature included, no free tier.
- **First customers:** adult children who subscribe for a parent's line, then protect their own.
- **Distribution:** banks, insurers and mobile operators bundle the plan for their customers.
- **To scale:** labelled French scam-call data, one distribution partner, and a consent model reviewed by the CNIL (the French data protection authority).

## Run it

Without a phone, the dashboard demo and the scenario replays need no accounts:

```bash
make install                                  # needs Python 3.12 and uv
make run                                      # then open http://localhost:8000/dashboard?demo=1
make replay SCENARIO=scenarios/scam_01_petit_fils.json
```

The full phone setup (Twilio number, AWS credentials, ngrok) is in [`docs/TECHNICAL.md`](docs/TECHNICAL.md).

## Limitations

- The scenarios are synthetic. They validate the pipeline, not real-world accuracy.
- Single user, single call: the contact book is a JSON file and live state is in memory.
- The web app only rings while it is open. A native app (CallKit / ConnectionService) is the next step.
