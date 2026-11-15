# Student guide

[Documentation](index.md) · [Project README](../README.md)

Samvaad is a place to practise speaking and review your reasoning. Its AI participants are not other students, interviewers, or examiners. Feedback is coaching guidance and can be wrong; it is not a hiring or academic decision.

## Before starting

Use a browser with microphone access, Web Audio, AudioWorklet, and WebSocket support, and a stable internet connection. Hosted microphone access needs a secure HTTPS page; the local development address also supports testing. Headphones help reduce echo. Device compatibility must be checked on your device; the repository does not establish a supported-browser list.

Read the site's **Data and privacy** page. Speech is sent to Gnani, and text context is sent to the configured reasoning service. Samvaad retains transcripts and feedback without storing raw-audio recordings. Avoid saying passwords, private documents, or confidential information.

## Create or access an account

1. Open the site and choose **Create account** or **Sign in**.
2. Enter an email address and a password between 10 and 128 characters.
3. After signing in, open the practice page.

The supplied configuration uses email/password without verifying email ownership. It does not offer automated forgotten-password recovery. Save your password securely. While signed in, choose **Account**, **Change password**, enter the new password, and select **Save password**. If the operator enables verified-email mode, the site instead exposes verification and email recovery flows.

There is no Google sign-in or self-service account/session deletion in this version. For data requests, use the contact shown on the site's privacy page. Account access cannot safely be restored using only a claim to an unverified email address.

## Choose a round

Open **Practice** and choose an available mode, topic, pressure, and duration. A mode marked unavailable cannot start. The five modes are Interview, Viva, Debate, Group discussion, and Presentation. The available durations are three and five minutes, subject to operator limits.

| Pressure | What to expect                                  |
| -------- | ----------------------------------------------- |
| Gentle   | More room to develop your explanation           |
| Standard | Direct follow-up questions and a fair challenge |
| Intense  | Firmer challenges to your reasoning             |

Group discussion uses an AI moderator and two AI peers. Other modes use one AI counterpart. A few setup and microphone-screen descriptions still describe an older group arrangement; the live room identifies the current participants.

Presentation protects the first two thirds of the round for delivery. Questions are allowed after that boundary. The timer includes the round's spoken opening.

Your daily allowance appears when available. Created rounds count towards the daily allowance, even if they do not reach a report. The daily allowance resets at midnight UTC, which is 5:30 a.m. in India. The operator also controls a shared monthly voice allowance.

## Check your microphone

1. Select **Test microphone**.
2. Allow the browser's microphone request.
3. Speak a sentence during the four-second test and check that the bars respond.
4. After the test reports success, select **Enter practice room**.
5. Select **Start discussion** in the room to connect.

The microphone test runs locally and does not save audio. It is separate from the voice session. A quiet or blocked input keeps room entry disabled until a successful test.

## During practice

The timer starts after the voice recognition connection is ready. A sleeping voice service may take time to wake; keep the page open while it connects. Speak in English and respond to the AI participants. You can speak over an AI response to interrupt it.

- **Mute microphone** stops your microphone contribution; the round's timer continues.
- **Mute speaker** silences the AI audio on your device; the round continues.
- **End round** opens a confirmation. Choose **Keep speaking** or confirm **End round**.

The voice room has no live captions, live score, or student waveform. It ends at the configured time. Finalizing the last speech and preparing feedback can take additional time.

Do not refresh or close the page during a live round. A disconnected round cannot resume. After a connection error, choose **Check round status** or return through **History** before starting another round.

## Read feedback

After the round, wait for **Read your scorecard**, or return through History. Processing can continue after you leave the feedback page; its timing depends on service availability and the operator's recovery schedule.

The report contains:

- Eight scores from 1 to 5: clarity, structure, relevance, evidence, counterargument handling, conciseness, composure, and closing.
- An overall score out of 100 calculated from those eight scores.
- Reasons and transcript quotes where reliable evidence is available.
- A strongest moment, an improvement opportunity, and exactly three practice actions.
- The transcript available for that completed round.

Select a dimension to read its reason and available quote. A highlighted transcript turn helps locate that evidence. Scores use recognized words; they do not judge accent, pronunciation, emotion, or vocal confidence. Recognition can mishear speech, and a reliable quote is not available for every dimension.

If there is too little verified conversation, an interrupted connection, or invalid generated feedback, the application may show a failed result instead. A missing score is not zero. A failed scoring attempt may still leave an available transcript on the status page; a failed live connection does not guarantee a finalized transcript.

## Repeat and review progress

Choose **Practise again** from a report to reuse its setup. Previous practice actions appear when the prior completed round belongs to your account and matches the selected mode and topic. These actions are shown as a practice reminder.

History lists your rounds, including those still processing or needing attention. Filter the list or choose **Load more rounds**. A progress chart appears only after at least two compatible completed rounds share the same mode, topic, rubric, pressure, and duration.

## Troubleshooting

| What you see                        | What to do                                                                                                                               |
| ----------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| Microphone access blocked           | Allow microphone access in the browser's site settings and test again.                                                                   |
| No sound during the test            | Check the selected microphone, hardware mute, and other applications using it.                                                           |
| Connection warming                  | Keep the page open; the practice timer has not started.                                                                                  |
| Interrupted connection              | Restore the connection, check the existing round in History, then start a new round if needed.                                           |
| No AI audio                         | Check speaker mute, device volume, browser audio permission, and headphones.                                                             |
| Feedback still preparing            | Return through History or choose Check for updates. Report persistent delays to the operator.                                            |
| Allowance used                      | Wait for the displayed daily reset or contact the operator about shared capacity.                                                        |
| Password forgotten while signed out | Automated recovery is unavailable under the supplied unverified-email configuration. Contact the operator without sharing your password. |

When reporting a problem, give the approximate time, device/browser, page, and visible message. Do not send passwords, access tokens, transcripts, or recordings in public reports.
