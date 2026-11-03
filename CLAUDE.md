# Climate Quest - project rules

## What it is
A single-file, iPhone-optimized HTML game for high school Earth Space Science in Florida. It has 8 chapters reached from an open menu. Chapter 1 (Ice Core Drill), Chapter 2 (Sunlight and Seasons), and Chapter 3 (Earth's Energy Balance) are done. Every chapter follows the same flow: narrated story, then interactive activity, then quiz. Quiz results go into a PDF the student shares to Canvas.

The whole game lives in `index.html` at the repo root.

## Hard technical rules (the game runs in Koder/WKWebView on iPhone)
- One self-contained HTML file. No libraries, CDNs, web fonts, image files, or network calls at runtime. Embed all data in the file.
- ES5 only: var and function. No arrow functions, template literals, let/const, classes, default parameters, for...of, spread, or destructuring.
- The whole file is ASCII only. Use '\u00B0' in JS and &deg; in HTML. No smart quotes, en dashes, or em dashes.
- No CSS custom properties (var(--x)). Use literal hex colors.
- Never call ctx.ellipse(). Use the CE(c,x,y,rx,ry) helper.
- No emoji in canvas fillText. Pass any user text drawn on canvas through clean().
- Show and hide elements with style.display from JS.
- Make sounds with WebAudio only (sfx object, tone()). No audio files. The Sound on/off switch lives in the Settings panel (#setCard); check V.on before playing.
- Must work in portrait on a 390x844 iPhone screen.

## Architecture (reuse what exists; don't duplicate)
- Screens: div.scr elements listed in the `screens` array. Switch with show(id), which stops loops and speech.
- Animation: startLoop(function(dt,t){...}). setupCanvas(cv, logicalW, logicalH, maxHeightFraction) returns a context scaled to logical coordinates.
- Story: startStory(STEPS, onDone, 'Last button label'). Each step is {sc:'sceneName', who:'frost'|'sam'|'rosa', tx:"..."}. Story canvas is 400x300 logical. Scenes chain drawScene -> drawScene2 -> drawScene3; add drawScene4, etc., each called from the previous one's final else.
- Characters: Dr. Frost is the scientist and narrator. Sam and Rosa are new workers who ask the questions students would ask. Narration uses say(text, WHO[who].pitch, who) so each character speaks in the voice chosen in Settings (the cog, top right). Mission and activity lines use say(text, 1.05), which defaults to Dr. Frost's voice.
- Quiz: startQuiz('chN', LIST). Question types:
  - {t:'mc', q, o:[4 options], a, why}
  - {t:'label', q, bg, labels:[{s,x,y,r} or {s,x,y,seg:[x1,y1,x2,y2],tol}]} on a 600x620 canvas
  - {t:'draw', q, bg, cols} on a 600x600 canvas
  - {t:'text', q}
- Register quiz backgrounds in drawBg(c,name). Every chapter quiz has 6 multiple choice, 1 label, 1 draw, and 1 typed answer.
- Menu: set ready:true on the chapter's CH entry and add a branch in openMenu's onclick to start it.
- The report and PDF already include any chapter saved in S.res. Don't change them.
- Missions: misRun(prefix, LIST, 'Last button label', onDone) runs the Check / Next mission flow (added in Chapter 3). It expects elements prefix+Head, Text, Check, Msg and a Hint button wired to misHint. Each mission is {tx, hint, ok:function(){}, yes, start:function(){} (optional)}.
- Helpers to reuse: circ, rr, txt, pill, star5, mix, sky, space, sunGlow, drawSide, drawOrbit, miniEarth, person, rig, rnd, fmt, lpos, cpos, el, angDiff, norm, mol (co2, ch4, h2o, n2, o2), molS (scaled mol), arw (arrow), wavyArw (infrared arrow), cloud, thermo, ebg (space/air/ground backdrop).

## Style and content
- Visuals: cute round cartoon characters with realistic-ish science visuals. Dark navy UI (#0E1B2C), orange buttons (#FF7A2F), ice blue (#BFE6F5), green for correct (#6FE3A4).
- Writing: narration lines are short and friendly, at an 8th to 10th grade reading level. Each activity has missions with hints and the Check / Next mission flow used in Chapter 2.
- Every science claim must be accurate. Use "about" for approximate numbers. If you are unsure of a fact or can't get real data, stop and ask me. Never invent data.
- Tie chapters back to earlier ones when it helps (ice cores, tilt, albedo).

## Workflow for every chapter
1. Read the latest `index.html` and build the chapter inside it.
2. Edit `index.html` in place. Older versions are kept in git history (no separate version files).
3. Before committing:
   - Run node --check on the extracted <script>.
   - Search the code (outside strings) for =>, backticks, let, const, ctx.ellipse, var(--, and any non-ASCII byte. Fix every hit.
4. If Playwright is available, click through story, activity, quiz, and PDF at 390x844. Fix any page errors and any text clipped at canvas edges.
5. Commit with a clear message, push, and tell me what changed.
