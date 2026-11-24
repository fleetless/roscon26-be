# 🐢 From a ROS 2 robot to an app

The handout of the workshop on 2026-11-25. In two blocks of 45 minutes, a
simulated turtle goes from `docker compose up` to a control page of your own,
on [Fleetless](https://fleetless.dev), and an agent does most of the typing.
Every command and every prompt below sits in its own block with a copy
button. Nothing here needs typing but your token and your address.

| Block | What happens | Ends with |
|---|---|---|
| 1 | Open an account, connect your agent, create a robot, start the turtle, let the agent write five entries of its configuration, look, publish | **Checkpoint 1**: the turtle drives from the console |
| 2 | Clone the app-starter, let the agent register the app, start it, let the agent build a control page, then hand it the turtle | Your page drives the turtle, and so does your agent |

## ✅ Before you start

- **Docker**: Docker Desktop on macOS or Windows, Docker Engine with Compose
  v2 on Linux. amd64 or arm64. No ROS, no Node, no GPU.
- **Git.**
- **An agent that speaks MCP.** The projector shows Claude Desktop in block 1
  and Claude Code in block 2. Codex, Cursor and the rest take the same URL.

Conference Wi-Fi is shared by everyone in the room, so two downloads go better
before you arrive. This repository:

```sh
git clone https://github.com/fleetless/roscon26-be.git
```

And the turtle, about 700 MB:

```sh
docker pull ghcr.io/fleetless/turtle-sim
```

## 1️⃣ Block 1: a robot that exposes nothing, then five things

**1. Open an account** at <https://console.fleetless.dev>. Sign-up is open,
and the free plan, Basic, holds one robot and one app: what today needs.

**2. Connect your agent.** Fleetless's endpoint for your team is
`https://mcp.fleetless.dev`. Your agent signs in as you and reaches what your
account reaches.

- **Claude Desktop.** Settings → Connectors → Add custom connector. Name it
  `Fleetless`, give it this URL, connect, and sign in with your Fleetless
  account:

  ```text
  https://mcp.fleetless.dev
  ```

- **Claude Code.** Signed in with the same claude.ai account, a connector
  added in Claude Desktop or on claude.ai is already there: `/mcp` lists it.
  Without one, for every folder at once:

  ```sh
  claude mcp add --transport http --scope user fleetless https://mcp.fleetless.dev
  ```

  Then `/mcp` inside Claude Code, and sign in.

- **Anything else.** Add the URL as a remote MCP server. Discovery and sign-in
  follow on their own.

The agent can move the robot. Today that is a turtle in a container. On a
real robot, a person watches every motion and can stop it at once.

**3. Create the robot.** In the console: **Robots** → **Connect robot**. Name
it `turtle`, then **Create robot**. The token, `frt_…`, is shown once: copy
it. Skip the install commands, the bridge is in the image. Tick **I saved the
token**, then **Next**; the dialog waits for the turtle. The token goes by
hand, not by agent: it has no business in a chat history.

**4. Start the turtle.** Open the folder `roscon26-be` you cloned in your
editor, VS Code for instance. In `turtle-sim/compose.yaml`, put your token in
place of `frt_...`:

```yaml
      FLEETLESS_TOKEN: frt_...
```

The token stays in that file on your machine: never in a chat, a commit or a
screenshot. Then, in a terminal in `roscon26-be`:

```sh
cd turtle-sim
```

```sh
docker compose up
```

**5. Watch it come online.** The view is at <http://localhost:8765>: the
garden from above, the turtle on its charging pad. Within seconds the console
says `turtle is online`. Its **Data** and **Controls** are empty: the robot
exposes nothing until its configuration says what.

Inside the container it is plain ROS 2. In a second terminal:

```sh
docker exec -it turtle ros2 topic list
```

**6. Ask the agent what is there.**

```text
Use the Fleetless tools, and call docs with action describe before you first use an action. Find my robot named turtle and read what its bridge reported about its ROS graph (console_introspection_read, action get). List its topics, services and actions with their types in one table. Change nothing.
```

**7. Let it write five entries.** The ROS graph knows topics and types. It
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

**8. Look, then publish.** In the console, open the turtle's
**Configuration**: the agent's draft. Check what the graph could not tell it:
the two speed limits, the percent, the 500 ms. Then **Publish**, and
**Publish** again in the dialog. The dialog lists one `unknown_type` warning
per entry: the cloud has not fetched this robot's message types yet. A
warning, not a refusal. **Data** and **Controls** fill within seconds.

### ✔️ Checkpoint 1

Everyone switches to the turtle's complete configuration, 500 lines, most of
them descriptions: they are all a model knows about the turtle, and block 2
builds on them. Open <http://localhost:8765/fleetless.yaml>; the browser saves
`turtle.fleetless.yaml`. On **Configuration**: **Upload fleetless.yaml**, the
arrow pointing up, pick the file, **Save draft**, **Publish**, **Publish**.

Then play on **Controls**:

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

## 2️⃣ Block 2: an app, a control page, an agent at the wheel

From here on, a coding agent in a terminal: Claude Code on the projector,
Codex or another just as well.

**1. Clone the app-starter.** It is a public template app on
`@fleetless/sdk`, not made for this workshop: the same repository starts any
app of yours, and GitHub's **Use this template** gives you a copy of your own.
Today it goes into `turtle-app`. In a terminal in `roscon26-be`:

```sh
git clone https://github.com/fleetless/app-starter.git turtle-app
```

```sh
cd turtle-app
```

**2. Start your agent in `turtle-app`.** It reads the template, its README
and its `AGENT-SETUP.md` from there.

**3. Let the agent register the app.** Put your own address in place of
`you@example.com`; the invitation goes to it.

```text
Read AGENT-SETUP.md and carry out the prompt under "The prompt" with the Fleetless MCP tools. Its step 1 is answered: (a) Turtle Control, (b) turtle_control, (c) http://localhost:3000, (d) the robot turtle, (e) you@example.com, (f) hand me the link, (g) no. If set_mcp is refused plan_required, skip it, since the app's MCP server needs Plus and today does not need it, and carry on with the next step. When it is done, copy .env.example to .env and set NUXT_PUBLIC_API_URL=https://api.fleetless.dev and NUXT_PUBLIC_APP_IDENTIFIER=turtle_control there, changing nothing else.
```

It reports an invitation link. Keep it.

**4. Start the app**, in a second terminal in `turtle-app`:

```sh
docker compose up -d
```

The first start installs about 1,100 packages into a Docker volume; later
starts skip that. Follow it until the log prints `Local:`, then Ctrl-C, which
leaves the app running:

```sh
docker compose logs -f app
```

Open the invitation link from step 3: a Fleetless page, where you set a
password. Then sign in at <http://localhost:3000>. The turtle is listed;
click it. Every tab on its page is drawn from its datasheet, the list of what
your role reaches.

**5. Let the agent build a control page.**

```text
Build a page in this app that controls my Fleetless robot turtle, and read robots.describe for it over MCP first to see what it offers. The app already runs in Docker (docker compose up -d) at http://localhost:3000 and reloads on every edit, so do not start it; this machine has no Node, so run pnpm only as docker compose exec app pnpm.
```

The page reloads as the agent writes. Drive the turtle from it.

**6. Hand the turtle to the agent.**

```text
Take the turtle as close to a pond as it gets, then back onto its charging pad. Call robots.describe for the robot turtle first and read the descriptions. After each step, tell me where the turtle is.
```

It reads the descriptions, gets `target_occupied` for a square of water,
picks a neighbour, and ends with `dock`. Nobody in this room wrote a server,
a database or a sign-in. That was the point.

## 🧹 Afterwards

In `turtle-app`, and then in `turtle-sim`:

```sh
docker compose down -v
```

The robot, the app and your app user stay in your Fleetless account until you
delete them in the console.

## 🔧 When something does not work

| You see | Because | Then |
|---|---|---|
| The turtle never turns online; the log says `invalid_token` | The token is wrong or missing | `turtle-sim/compose.yaml`: `FLEETLESS_TOKEN:`, a space, the token, no quotes; Ctrl-C and `docker compose up` again |
| `unknown_type` warnings on publish | The cloud has not fetched the turtle's message types | Nothing; or the first prompt below |
| Every call of the app fails as a network error | The app's origin is not in its `allowed_origins` | The second prompt below |
| The app says "That does not exist, or you cannot reach it." | `NUXT_PUBLIC_APP_IDENTIFIER` is not the app's identifier | Fix `.env`, then `docker compose up -d` again |
| Port 3000 is taken | Something else runs there | `PORT=3100` in `.env`, and the second prompt below with `http://localhost:3100` |
| An edit does not reach the browser (Windows) | Docker Desktop passes no file events from a Windows drive | `CHOKIDAR_USEPOLLING=true` in `.env`, then `docker compose up -d` again |
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
