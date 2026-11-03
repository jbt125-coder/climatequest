# Climate Quest - project rules

## What it is
A single-file, iPhone-optimized HTML game for high school Earth Space Science in Florida. It has 8 chapters reached from an open menu. All 8 chapters are done: Ice Core Drill, Sunlight and Seasons, Earth's Energy Balance, Orbit Cycles and Ice Ages, Moving Air, Mountains and Rain Shadows, City Planner, and Climate Spiral. Every chapter follows the same flow: narrated story, then interactive activity, then quiz. Quiz results go into a PDF the student shares to Canvas.

The whole game lives in `index.html` at the repo root.

## Hard technical rules (the game runs in Koder/WKWebView on iPhone)
- One self-contained HTML file. No libraries, CDNs, web fonts, image files, or network calls at runtime. Embed all data in the file.
- ES5 only: var and function. No arrow functions, template literals, let/const, classes, default parameters, for...of, spread, or destructuring.
- The whole file is ASCII only. Use '\u00B0' in JS and &deg; in HTML. No smart quotes, en dashes, or em dashes.
- Chemical formulas always use correct subscripts (CO2 is written with a subscript 2, also H2O, CH4, N2O, N2, O2). In HTML use <sub>2</sub>. In JS strings use the escapes '\u2080' to '\u2089' (for example 'CO\u2082'). txt(), pill(), quiz label tags, and the PDF draw these as small lowered digits through ftext()/mtext(), so draw canvas text through txt()/pill()/ftext(), not raw fillText. say() turns them back into plain digits for speech. Student-typed text is left as typed.
- No CSS custom properties (var(--x)). Use literal hex colors.
- Never call ctx.ellipse(). Use the CE(c,x,y,rx,ry) helper.
- No emoji in canvas fillText. Pass any user text drawn on canvas through clean().
- Show and hide elements with style.display from JS.
- Make sounds with WebAudio only (sfx object, tone()). No audio files. The Sound on/off switch lives in the Settings panel (#setCard); check V.on before playing.
- Must work in portrait on a 390x844 iPhone screen.

## Architecture (reuse what exists; don't duplicate)
- Screens: div.scr elements listed in the `screens` array. Switch with show(id), which stops loops and speech.
- Animation: startLoop(function(dt,t){...}). setupCanvas(cv, logicalW, logicalH, maxHeightFraction) returns a context scaled to logical coordinates.
- Story: startStory(STEPS, onDone, 'Last button label'). Each step is {sc:'sceneName', who:'frost'|'sam'|'rosa', tx:"..."}. Story canvas is 400x300 logical. Scenes chain drawScene -> drawScene2 -> drawScene3 -> drawScene4 -> drawScene5 -> drawScene6 -> drawScene7 -> drawScene8 (drawScene6 sends names starting with sl_ to the Chapter 6 slides; drawScene8 sends ev1 to ev12 to the Chapter 8 evidence slides). slideFrame(c,t,name,title,order) numbers slides from the order list you pass., each called from the previous one's final else.
- Characters: Dr. Frost is the scientist and narrator. Sam and Rosa are new workers who ask the questions students would ask. Narration uses say(text, WHO[who].pitch, who) so each character speaks in the voice chosen in Settings (the cog, top right). Voices are limited to Daniel, Samantha, and Karen (VSUG); defaults are Dr. Frost = Daniel, Sam = Samantha, Rosa = Karen (VDEF), and any character can be switched to any of the three. Mission and activity lines use say(text, 1.05), which defaults to Dr. Frost's voice.
- Quiz: startQuiz('chN', LIST). Question types:
  - {t:'mc', q, o:[4 options], a, why}
  - {t:'label', q, bg, labels:[{s,x,y,r} or {s,x,y,seg:[x1,y1,x2,y2],tol}]} on a 600x620 canvas
  - {t:'draw', q, bg, cols} on a 600x600 canvas
  - {t:'text', q}
- Register quiz backgrounds in drawBg(c,name). Every chapter quiz has 6 multiple choice, 1 label, 1 draw, and 1 typed answer.
- Menu: set ready:true on the chapter's CH entry and add a branch in openMenu's onclick to start it.
- Results screen (openReport): one "Chapter N - Title PDF Results" button per finished chapter in S.res, plus "All finished chapters in one PDF". Each chapter becomes 1 page; all chapters fit on 2. Students save it as a PDF or as a picture (JPEG). New chapters appear here automatically; don't change this code when adding a chapter. buildPages(only, maxPages, title) lays results out in two columns, trying 160% size first and shrinking (down to a 70% floor) until it fits; below the floor it adds pages.
- Sign-in: first and last name plus period, confirmed then locked. Each result in S.res stores the name/period it was finished under (resCur, repWho); a name change never rewrites old PDFs. Quiz scores under 70% show a Try again button.
- Chapter starts go through startChapter(i) (headphones card) then launchChapter(i). Stories have no Skip; Next unlocks after each line is spoken (say() callback) plus a minimum reading time, with a backup timer.
- Settings (#setCard) has a teacher code box: 'yourdadacademy' clears the camp cooldown (S.campNext=0). The menu has no name-change link; a name is locked once confirmed.
- Camp break mini game (scrCamp, campStart/campUpdate/campDraw): reached from the menu after sign-in. Side view of the camp: feed penguins and Weddell seals (breathing holes in C_HOLES) by tossing fish to them. Fish bounce and slide on the ice (cFishStep) and feed any waiting animal they touch, in the air or sliding; a fish that slides into an empty breathing hole is lost. A glacier cliff with an overhang sits on the right (C_LX, C_BX, C_CT, C_LB); Dr. Frost can't climb it, and penguins on top (a.top) are worth +2 and need a jump shot. Movement, jumping, and the fish arc copy Basketball Builder's physics, and the controller is its d-pad plus A (jump, double jump) and B (hold for the power meter, release to toss); keys arrows/WASD, Z, Space. Feeding +1 heart (+2 after bouncing off a skua), bopping a skua +1, poop or a swooping skua -1 (never below 0). 1 minute, 10-minute cooldown in S.campNext, best in S.campBest. person() takes o.up for arms raised.
- Label tasks can set guide:true to draw dashed target circles at each label's x,y. Use seg bands for labels that name a whole layer or region so students don't have to hit one exact spot.
- Missions: misRun(prefix, LIST, 'Last button label', onDone) runs the Check / Next mission flow (added in Chapter 3). It expects elements prefix+Head, Text, Check, Msg and a Hint button wired to misHint. Each mission is {tx, hint, ok:function(){}, yes, start:function(){} (optional)}.
- Helpers to reuse: circ, rr, txt, pill, star5, mix, sky, space, sunGlow, drawSide, drawOrbit, miniEarth, person, rig, rnd, fmt, lpos, cpos, el, angDiff, norm, mol (co2, ch4, h2o, n2, o2), molS (scaled mol), arw (arrow), wavyArw (infrared arrow), cloud, thermo, ebg (space/air/ground backdrop), ovalOrbit (animated elliptical orbit), tGlobe (globe with adjustable tilt and 65 N line), wobTop (precessing axis), naMap (approximate North America map with optional ice sheet), icGraph, mapXY (lon/lat to naMap pixels), cellGlobe (schematic globe with Hadley/Ferrel/Polar cells and wind belts), cellLoop, frontLine (cold/warm front symbols), amTag (air mass tag), AM_REG (air mass source regions), tankScene, crossSec (mountain cross-section with wet/dry sides), rainCloud, slideFrame/slideLabel (slide-style story frame used in Chapter 6), worldMap/wmXY (approximate world map with equator, tropics, Arctic Circle), climo/climoGeom (climograph), kop/kopWhy (textbook Koppen group and explanation from data), tF (C with F), mmIn (mm with inches).
- Never pass mix() output into mix() again: mix() only reads #rrggbb hex colors.
- Real data already embedded: IC_T (Dome C Antarctic temperature change x10, Jouzel et al. 2007) and IC_C (CO2 ppm, Bereiter et al. 2015), both 800 values of 1,000-year averages from NOAA NCEI, index 0 = most recent, CO2 ending about 1850. Reuse them instead of re-embedding.
- CITY: 10 cities of monthly normals (hi, lo, av in C; pr in mm). 8 are NOAA NCEI 1991-2020 (Miami, Phoenix, Orlando, San Francisco, Chicago, Fairbanks, Utqiagvik), Singapore is NEA 1991-2020, London is Met Office Kew Gardens 1991-2020, and Quito is older WMO averages (period not stated). Los Angeles was dropped because its 1991-2020 data classifies as B, not Mediterranean C.
- GT_RAW / GT: NASA GISTEMP v4 monthly global anomalies from Jan 1880 (uploaded file, through Sep 2026), shown relative to 1880-1900 (GT_BASE). GT_ANN = yearly averages; GT_WARM, GT_FIRST1, GT_10 are computed from the data. To update, replace GT_RAW from a newer GLB.Ts+dSST.csv; all story and quiz numbers recompute.
- Data downloads: this environment blocks www.ncei.noaa.gov and data.giss.nasa.gov. If a chapter needs data, ask the user to upload the file in chat.

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
