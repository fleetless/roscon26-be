# 🐢 From a ROS 2 robot to an app

The handout of the workshop on 2026-11-25. In two blocks of 45 minutes, a
simulated turtle goes from `docker run` to a control page of your own, on
[Fleetless](https://fleetless.dev), and an agent does most of the typing.
Every command and every prompt below sits in its own block with a copy
button. Nothing here needs typing but your own addresses; the token you
paste.

| Block | What happens | Ends with |
|---|---|---|
| 1 | Create a robot, start the turtle, let an agent write five entries of its configuration, look, publish | **Checkpoint 1**: the turtle drives from the console |
| 2 | Let the agent set up an app, start the app-starter, let it build a control page, invite a second person | **Checkpoint 2**: your page drives the turtle; the second person sees it and cannot |

## ✅ Before you start

- **Docker**: Docker Desktop on macOS or Windows, Docker Engine with Compose
  v2 on Linux. amd64 or arm64. No ROS, no Node, no GPU.
- **Git.**
- **An agent that speaks MCP.** The projector shows Claude Desktop in block 1
  and Claude Code in block 2. Cursor, Codex and the rest take the same URL.
- **A Fleetless account** at <https://console.fleetless.dev>. The free plan,
  Basic, holds one robot and one app, which is what today needs.

Conference Wi-Fi is shared by everyone in the room. The downloads go better
before you arrive. The turtle:

```sh
docker pull ghcr.io/fleetless/turtle-sim
```

The app-starter and its dependencies, about 1,100 packages: clone it, start
it once, and press Ctrl-C when the log prints `Local:`.

```sh
git clone https://github.com/fleetless/app-starter.git
```

```sh
cd app-starter
```

```sh
docker compose up
```

## 🔌 Connect your agent

Fleetless's endpoint for your team is `https://mcp.fleetless.dev`. Your agent
signs in as you and reaches what your account reaches.

**Claude Desktop.** Settings → Connectors → Add custom connector. Name it
`Fleetless`, give it this URL, connect, and sign in with your Fleetless
account:

```text
https://mcp.fleetless.dev
```

**Claude Code.** Signed in with the same claude.ai account, a connector
added in Claude Desktop or on claude.ai is already there: `/mcp` lists it.
Without one, for every folder at once:

```sh
claude mcp add --transport http --scope user fleetless https://mcp.fleetless.dev
```

Then `/mcp` inside Claude Code, and sign in.

**Anything else.** Add the URL as a remote MCP server. Discovery and sign-in
follow on their own.

The agent can move the robot. Today that is a turtle in a container. On a
real robot, a person watches every motion and can stop it at once.

## 1️⃣ Block 1: a robot that exposes nothing, then five things

**1. Create the robot.** In the console: **Robots** → **Connect robot**. Name
it `turtle`, then **Create robot**. The token, `frt_…`, is shown once: copy
it. Skip the install commands, the bridge is in the image. Tick **I saved the
token**, then **Next**; the dialog waits for the turtle. The token goes by
hand, not by agent: it has no business in a chat history.

**2. Start the turtle.** Make a file `turtle.env` in the folder you run
Docker from, with this one line, your token in place of `frt_...`:

```text
FLEETLESS_TOKEN=frt_...
```

Then, the same line in bash, PowerShell and cmd:

```sh
docker run --rm -it --name turtle -p 127.0.0.1:8765:8765 -v turtle-state:/state --env-file turtle.env ghcr.io/fleetless/turtle-sim
```

The view is at <http://localhost:8765>: the garden from above, the turtle on
its charging pad. Within seconds the console says `turtle is online`. Its
**Data** and **Controls** are empty: the robot exposes nothing until its
configuration says what.

Inside the container it is plain ROS 2. In a second terminal:

```sh
docker exec -it turtle ros2 topic list
```

**3. Ask the agent what is there.** In Claude Desktop:

```text
Use the Fleetless tools, and call docs with action describe before you first use an action. Find my robot named turtle and read what its bridge reported about its ROS graph (console_introspection_read, action get). List its topics, services and actions with their types in one table. Change nothing.
```

**4. Let it write five entries.** The ROS graph knows topics and types. It
does not know limits, units or picture sizes; those are in the prompt,
because only someone who knows the robot can say them.

```text
Write a draft fleetless.yaml for the robot turtle with exactly these five entries, built from the ROS graph you just read. Call docs with action describe for console_config_write before you use it.

- datapoint battery_soc: the battery's charge from /battery_state, in percent from 0 to 100 (the message carries 0.0 to 1.0), kept as history, with an alert battery_low below 20 %.
- datapoint square: the square the turtle is in, from /turtle/status.
- publisher drive: velocity commands on /cmd_vel. linear_speed at most 0.5 m/s either way, angular_speed at most 2.0 rad/s either way. When no command has arrived for 500 ms, the bridge sends a zero Twist.
- camera top: from /turtle/camera/top/image_raw, 480 x 480 pixels at 10 fps.
- action drive_to_square: /turtle/drive_to_square, whose goal is a square from A1 to P16.

Give every entry a description a model can act on. Save it with console_config_write, action draft_put, and do not publish. Then show me the YAML and the findings the draft returned.
```

**5. Look, then publish.** In the console, open the turtle's
**Configuration**: the agent's draft. Check what the graph could not tell it:
the two speed limits, the percent, the 500 ms. Then **Publish**, and
**Publish** again in the dialog. The dialog lists one `unknown_type` warning
per entry: the cloud has not fetched this robot's message types yet. A
warning, not a refusal. **Data** and **Controls** fill within seconds.

### ✔️ Checkpoint 1

Everyone switches to the turtle's complete configuration, 500 lines, most of
them descriptions: they are all a model knows about the turtle. Open
<http://localhost:8765/fleetless.yaml>; the browser saves
`turtle.fleetless.yaml`. On **Configuration**: **Upload fleetless.yaml**, the
arrow pointing up, pick the file, **Save draft**, **Publish**, **Publish**.

Then, on **Controls**:

- **Send** on `drive` with `linear_speed` `0.5`: half a second, a quarter of
  a metre, then the turtle stops by itself.
- **Run** on `drive_to_square` with `H9`: a job. It ends `succeeded`, reason
  `reached`.
- `M5`: `target_occupied`, the square is all pond. `N13`: `unreachable`, a
  courtyard without a door.
- **Run** on `dock`, then switch off Wi-Fi. The view at
  <http://localhost:8765> runs on your laptop: the turtle drives on to its
  pad. Switch Wi-Fi back on, and the job's result, `docked`, arrives in the
  console.

## 2️⃣ Block 2: an app, a control page, a second person

From here on, Claude Code in a terminal.

**1. Clone the app-starter**, a template app on `@fleetless/sdk`, unless you
did before you came:

```sh
git clone https://github.com/fleetless/app-starter.git
```

```sh
cd app-starter
```

Start Claude Code in that folder.

**2. Let the agent set up the app.** The prompt is the one in the
app-starter's
[`AGENT-SETUP.md`](https://github.com/fleetless/app-starter/blob/main/AGENT-SETUP.md#the-prompt);
copy it from there. It asks seven things once:

| Question | Answer |
|---|---|
| (a) app name | `Turtle Control` |
| (b) app identifier | `turtle_control` |
| (c) origin | `http://localhost:3000` |
| (d) robots | `turtle` |
| (e) first app user | your own address |
| (f) mail or link | the link |
| (g) links open in this app | no |

On Basic, the step `set_mcp` is refused `plan_required`: an app's own MCP
server comes with Plus. Today needs none, because step 7 uses the central
endpoint. The prompt stops at a refusal and asks; answer:

```text
Skip set_mcp: the app's MCP server needs Plus, and today does not need it. Continue with step 4.
```

It reports a line for `.env` and an invitation link. Keep the link.

**3. Point the app at Fleetless.**

```text
Copy .env.example to .env. In .env, set NUXT_PUBLIC_API_URL=https://api.fleetless.dev and NUXT_PUBLIC_APP_IDENTIFIER=turtle_control. Change nothing else.
```

**4. Start the app**, in a second terminal in the same folder:

```sh
docker compose up
```

The first start installs the dependencies into a Docker volume; later starts
skip that. Open the invitation link from step 2: a Fleetless page, where you
set a password. Then sign in at <http://localhost:3000>. The turtle is listed;
click it. Every tab on its page is drawn from its datasheet, the list of what
your role reaches.

**5. Let the agent build a control page.**

```text
Add a Control tab to the robot page of this app. First read README.md, app/utils/datasheet.ts, app/pages/robots/[id].vue, app/components/robot/CameraPanel.vue and the SDK reference at https://docs.fleetless.dev/reference/sdk/. Call robots.describe for the robot turtle over MCP to see its slugs and their descriptions.

The tab holds four parts:
1. Map. Download https://raw.githubusercontent.com/fleetless/turtle-sim/main/maps/garden.png into public/. It is 256 x 256 pixels, one pixel per cell, north up. Draw the turtle on it as an arrow, live: its cell from the datapoints cell_x and cell_y (0 to 255, counted from the south-west corner, so y grows upwards), its heading from heading (degrees, 0 is east, counter-clockwise).
2. Camera. The camera top, with the existing CameraPanel component.
3. Joystick. While it is held, publish drive with linear_speed (-0.5 to 0.5) and angular_speed (-2.0 to 2.0) every 100 ms. On release, stop publishing: the bridge stops the turtle 500 ms after the last message.
4. Stop. Stop publishing, then cancel whatever job runs on drive_to_square, drive_to and dock. A refusal because nothing runs there is fine.

Show each part only when the datasheet lists what it needs: joystick and stop the publisher drive, the map the datapoints cell_x, cell_y and heading, the camera the camera top. Keep every existing tab. Add no dependency: this machine has no Node, and every pnpm command runs as docker compose exec app pnpm. When done, run docker compose exec app pnpm lint and docker compose exec app pnpm typecheck, and fix what they report.
```

The page reloads as the agent writes. Drive the turtle from it. Hold the
joystick and switch off Wi-Fi: half a second later the turtle stands. That
stop is the bridge's, on the robot, not the app's.

**6. Invite a second person.** Put a second address of yours in place of the
one below; `you+observer@…` works with most providers. No mail goes out, so
it only has to be yours on paper.

```text
Invite an app user to the app turtle_control: the address you+observer@example.com, with the role observe. Do not send mail; give me the accept link.
```

Open the link in a private window, set a password, and sign in at
<http://localhost:3000> there.

### ✔️ Checkpoint 2

The private window shows the same Control tab with the map and the camera,
and no joystick and no stop. The role `observe` reaches no publisher, so
`drive` is not in its datasheet, and the page draws nothing for it. Nothing
in the app decided that; the role did.

**7. Hand the turtle to the agent.** In Claude Code or Claude Desktop:

```text
Take the turtle as close to a pond as it gets, then back onto its charging pad. Call robots.describe for the robot turtle first and read the descriptions. After each step, tell me where the turtle is.
```

It reads the descriptions, gets `target_occupied` for a square of water,
picks a neighbour, and ends with `dock`. Nobody in this room wrote a server,
a database or a sign-in. That was the point.

## 🧹 Afterwards

Ctrl-C stops the turtle, and `--rm` removes its container. In `app-starter`:

```sh
docker compose down -v
```

```sh
docker volume rm turtle-state
```

The robot, the app and both users stay in your Fleetless account until you
delete them in the console.

## 🔧 When something does not work

| You see | Because | Then |
|---|---|---|
| The turtle never turns online | The token is wrong or missing | `turtle.env`: one line, no quotes, no spaces |
| `unknown_type` warnings on publish | The cloud has not fetched the turtle's message types | Nothing; or the first prompt below |
| Every call of the app fails as a network error | The app's origin is not in its `allowed_origins` | The second prompt below |
| The app says "That does not exist, or you cannot reach it." | `NUXT_PUBLIC_APP_IDENTIFIER` is not the app's identifier | Fix `.env`, then `docker compose up` again |
| Port 3000 is taken | Something else runs there | `PORT=3100` in `.env`, and the second prompt below with `http://localhost:3100` |
| An edit does not reach the browser (Windows) | Docker Desktop passes no file events from a Windows drive | `CHOKIDAR_USEPOLLING=true` in `.env`, then `docker compose up` again |
| Your console sign-in does not sign you in to the app | App users and Fleetless users are separate accounts | Sign in with the password you set on the invitation |

Fetch the message types:

```text
Fetch the message types of the robot turtle with console_introspection_write, action fetch_types, then publish its configuration again.
```

Allow the app's origin:

```text
Add http://localhost:3000 to the allowed_origins of the app turtle_control, keeping everything else in its registration slice as it is.
```

## 📄 License

MIT, see [`LICENSE`](LICENSE). The turtle is
[turtle-sim](https://github.com/fleetless/turtle-sim), the app is the
[app-starter](https://github.com/fleetless/app-starter), and the rest is
in the [Fleetless docs](https://docs.fleetless.dev).
