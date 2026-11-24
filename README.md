# 🐢 From a ROS 2 robot to an app

This is the handout for the workshop on 25 November 2026. Over two blocks of
45 minutes you connect a simulated turtle to [Fleetless](https://fleetless.dev)
and build your own app to control it. A coding agent does most of the
typing for you.

You don't need to copy anything from the projector. Every command and every
prompt is in its own box below, and the copy button in the corner of each box
takes the whole thing. The only things you fill in yourself are your robot
token and your email address.

| Block | What you do | Where you end up |
|---|---|---|
| 1 | Sign up, connect your agent, add the turtle, let the agent write its first configuration and drive with it | The turtle runs on its complete configuration, and you drive it from the console |
| 2 | Clone the app template, let the agent register the app, then build the app you want together with it | An app of your own that drives the turtle |

## ✅ Before you start

You need:

- **Docker.** Docker Desktop on macOS or Windows, or Docker Engine with
  Compose v2 on Linux. Both amd64 and arm64 work. You don't need ROS, Node or
  a GPU.
- **Git.**
- **A coding agent that supports MCP**, for example Claude Desktop, Claude
  Code, Codex or Cursor. On the projector you'll see Claude Desktop in block 1
  and Claude Code in block 2.

The conference Wi-Fi is shared by everyone, so please download two things
before you arrive. First this repository:

```sh
git clone https://github.com/fleetless/roscon26-be.git
```

Then the turtle simulator image (about 700 MB):

```sh
docker pull ghcr.io/fleetless/turtle-sim
```

## 1️⃣ Block 1: Connect the turtle

**1. Create a Fleetless account** at <https://console.fleetless.dev>. Sign-up
is open to everyone. The free Basic plan includes one robot and one app,
which is all you need today.

**2. Connect your agent to Fleetless.** Your agent talks to Fleetless through
the MCP server at `https://mcp.fleetless.dev`. It signs in with your account
and can see and change everything your account can.

- **Claude Desktop:** go to Settings → Connectors → Add custom connector,
  name it `Fleetless`, paste the URL below, click connect and sign in with
  your Fleetless account.

  ```text
  https://mcp.fleetless.dev
  ```

- **Claude Code:** if you're signed in with the same claude.ai account and
  already added the connector in Claude Desktop or on claude.ai, it's there
  already. Type `/mcp` to check. Otherwise add it for all folders with:

  ```sh
  claude mcp add --transport http --scope user fleetless https://mcp.fleetless.dev
  ```

  Then type `/mcp` in Claude Code and sign in.

- **Other agents:** add the URL as a remote MCP server. The agent should find
  the sign-in by itself.

Keep in mind that the agent can move the robot. Today it's only a turtle in a
container. With a real robot, someone should always be watching and ready to
stop it.

**3. Add the robot in the console.** Go to **Robots** → **Connect robot**,
name it `turtle` and click **Create robot**. The console now shows a token
starting with `frt_`. Copy it, because it is only shown once. You can skip
the install commands, since the bridge is already part of the turtle image.
Tick **I saved the token** and click **Next**. The dialog then waits for the
turtle to connect.

We do this step by hand on purpose: the token is a password for your robot
and shouldn't end up in a chat with your agent.

**4. Start the turtle.** Open the `roscon26-be` folder you cloned in your
editor, for example VS Code. Open `turtle-sim/compose.yaml` and replace
`frt_...` with your token:

```yaml
      FLEETLESS_TOKEN: frt_...
```

Keep the token in this file on your machine. Don't paste it into a chat,
commit it or show it in a screenshot. Then open a terminal in `roscon26-be`
and start the turtle in the background:

```sh
cd turtle-sim
```

```sh
docker compose up -d
```

The terminal is free again right away. If you want to see what the turtle is
doing, follow its log, and press Ctrl-C when you've seen enough. The turtle
keeps running.

```sh
docker compose logs -f turtle
```

**5. Check that it's online.** Open <http://localhost:8765> to see the
garden from above, with the turtle on its charging pad. After a few seconds
the console shows `turtle is online`. The **Data** and **Controls** tabs are
still empty. That's expected: Fleetless shows nothing from a robot until you
configure what it should expose.

The container runs a normal ROS 2 system. If you're curious, run this:

```sh
docker exec -it turtle ros2 topic list
```

**6. Ask the agent what the robot has.** Paste this prompt into your agent:

```text
Use the Fleetless tools, and call docs with action describe before
you first use an action. Find my robot named turtle and ask its
bridge for its current ROS graph (console_introspection_write,
action refresh). List its topics, services and actions with their
types in one table. Change nothing else.
```

**7. Let the agent write a first configuration.** The ROS graph tells the
agent which topics exist and what types they have. It can't know speed
limits, units or image sizes, so the prompt gives them. That's the part only
someone who knows the robot can fill in.

```text
Write a draft fleetless.yaml for the robot turtle with exactly these
six entries, built from the ROS graph you just read. Call docs with
action describe for console_config_write before you use it.

- datapoint battery_soc: the battery's charge from /battery_state,
  in percent from 0 to 100 (the message carries 0.0 to 1.0), kept as
  history, with an alert battery_low below 20 %.
- datapoint square: the square the turtle is in, from /turtle/status.
- publisher drive: velocity commands on /cmd_vel. linear_speed at
  most 0.5 m/s either way, angular_speed at most 2.0 rad/s either way.
  When no command has arrived for 500 ms, the bridge sends a zero
  Twist.
- camera top: from /turtle/camera/top/image_raw, 480 x 480 pixels at
  10 fps.
- action drive_to_square: /turtle/drive_to_square, whose goal is a
  square from A1 to P16.
- action dock: /turtle/dock, which drives the turtle onto its
  charging pad in square A1.

Give every entry a description a model can act on. Save it with
console_config_write, action draft_put, and do not publish. Then show
me the YAML and the findings the draft returned.
```

**8. Review and publish.** In the console, open the turtle's
**Configuration** tab. You'll find the agent's draft there. Check the values
the agent couldn't have known: the two speed limits, the percentage and the
500 ms. If they look right, click **Publish** and confirm with **Publish** in
the dialog. The dialog shows an `unknown_type` warning for each entry,
because Fleetless hasn't loaded the turtle's message types yet. It's only a
warning and you can publish anyway. A few seconds later **Data** and
**Controls** fill up.

**9. Let the agent drive.** Back to your agent. It can only use the six
entries it wrote, so the prompt tells it where the pond is. It also tells
the agent that you've already published its draft, so it doesn't try to
do that itself.

```text
I reviewed your draft and published it in the console, so its six
entries are live on the turtle now. Do not change the configuration.
Call robots.describe for the robot turtle, then take the turtle as
close to the pond in square M5 as it gets, and back onto its charging
pad. After each step, tell me where the turtle is.
```

Watch it in the view at <http://localhost:8765>. M5 is all water, so
`drive_to_square` answers `target_occupied`, and the agent has to pick a
square next to it on its own. Some of those are water too, so two or three
refusals before it gets there are normal. At the end it sends the turtle
home with `dock`. The six entries and their descriptions are all it has to
go on.

### ✔️ Checkpoint 1

Now everyone loads the turtle's complete configuration. It's about 500 lines,
and most of it is descriptions, because that's everything an AI model will
know about the turtle. Block 2 builds on it.

Open <http://localhost:8765/fleetless.yaml>. Your browser saves it as
`turtle.fleetless.yaml`. In the console, on **Configuration**, click
**Upload fleetless.yaml** (the arrow pointing up), pick the file, then
**Save draft**, **Publish** and **Publish**.

Then try things out on **Controls**:

- **Send** on `drive` with `linear_speed` set to `0.5`. The turtle moves for
  half a second, about a quarter of a metre, and stops by itself.
- **Run** on `drive_to_square` with `H9`. This starts a job, which ends with
  `succeeded` and the reason `reached`.
- Try `M5` and `N13` as well. `M5` is in the pond, so you get
  `target_occupied`. `N13` is a courtyard without a door, so you get
  `unreachable`.
- **Run** on `dock`, then switch off your Wi-Fi. The view at
  <http://localhost:8765> runs on your laptop, so you can watch the turtle
  keep driving to its pad. Switch the Wi-Fi back on, and the result,
  `docked`, shows up in the console.

## 2️⃣ Block 2: Build your own app

For this block you use a coding agent in a terminal. We'll use Claude Code on
the projector, but Codex or any other agent works too.

**1. Clone the app template.** The app-starter is a public template for
building apps on `@fleetless/sdk`. It isn't specific to this workshop, so you
can use it for your own projects later, for example with GitHub's
**Use this template** button. Today we clone it into a folder called
`turtle-app`. In a terminal in `roscon26-be`:

```sh
git clone https://github.com/fleetless/app-starter.git turtle-app
```

```sh
cd turtle-app
```

**2. Start your agent in the `turtle-app` folder.** That way it has the whole
template as context, including the README and `AGENT-SETUP.md`.

**3. Let the agent register the app with Fleetless.** Replace
`you@example.com` with your own email address before you send the prompt.
That's where the invitation for your app account goes.

```text
Read AGENT-SETUP.md and carry out the prompt under "The prompt" with
the Fleetless MCP tools. Its step 1 is answered:
(a) Turtle Control
(b) turtle_control
(c) http://localhost:3000
(d) the robot turtle
(e) you@example.com
(f) hand me the link
(g) no.
If set_mcp is refused plan_required, skip it, since the app's MCP
server needs Plus and today does not need it, and carry on with the
next step. When it is done, copy .env.example to .env and set
NUXT_PUBLIC_API_URL=https://api.fleetless.dev and
NUXT_PUBLIC_APP_IDENTIFIER=turtle_control there, changing nothing
else.
```

When the agent is done, it gives you an invitation link. Keep it for step 4.

**4. Start the app.** Open another terminal in `turtle-app` and run:

```sh
docker compose up -d
```

The first start downloads about 1,100 packages, which takes a few minutes.
Later starts are quick. To see when it's ready, follow the log until it
shows `Local:`, then press Ctrl-C. The app keeps running in the background.

```sh
docker compose logs -f app
```

Now open the invitation link from step 3 and set a password. Then sign in at
<http://localhost:3000>. You'll see the turtle in the list. Click on it: the
tabs on its page are generated from what your role is allowed to see and do.

**5. Let the agent build a first control page.**

```text
Build a page in this app that controls my Fleetless robot turtle, and
read robots.describe for it over MCP first to see what it offers.
The app already runs in Docker (docker compose up -d) at
http://localhost:3000 and reloads on every edit, so do not start it;
this machine has no Node, so run pnpm only as
docker compose exec app pnpm.
```

The page in your browser updates while the agent works. Once it's done, try
driving the turtle from your new page.

**6. Make it yours.** From here on there's no fixed goal. Keep working with
your agent on the app you'd like to have, one small request at a time, and
look at the result in the browser after each one. A few ideas if you need a
place to start:

- Put the `top` or `eye` camera on the page.
- Draw the garden as a grid and send the turtle to a square by clicking it,
  or put it there at once with `teleport`.
- Drive with the arrow keys.
- Show the battery's history as a chart, and a warning when it drops below
  20 %.
- Drop rocks in the turtle's way with `set_block` and watch it plan around
  them. `reset` clears the garden again.
- A stop button that cancels whatever job is running.

If something is refused, paste the error into the chat. The agent can read
the docs and the robot's configuration and usually works out why.

Over the whole workshop nobody wrote a server, a database or a login. All of
that came from Fleetless.

## 🧹 Afterwards

To stop everything, run this in `turtle-app` and then again in `turtle-sim`:

```sh
docker compose down -v
```

Your robot, your app and your app user stay in your Fleetless account until
you delete them in the console.

## 🔧 Troubleshooting

| Problem | Cause | What to do |
|---|---|---|
| The turtle doesn't come online and `docker compose logs turtle` shows `invalid_token` | The token is wrong or missing | Check `FLEETLESS_TOKEN:` in `turtle-sim/compose.yaml`: a space after the colon, then the token, no quotes. Then run `docker compose up -d` again |
| `unknown_type` warnings when publishing | Fleetless hasn't loaded the turtle's message types yet | You can ignore them, or use the first prompt below |
| Every request from the app fails with a network error | The app's address is missing from its `allowed_origins` | Use the second prompt below |
| The app says "That does not exist, or you cannot reach it." | `NUXT_PUBLIC_APP_IDENTIFIER` in `.env` doesn't match the app | Fix `.env`, then run `docker compose restart app` |
| Port 3000 is already in use | Another program uses it | Add `PORT=3100` to `.env`, run `docker compose up -d` again, and use the second prompt below with `http://localhost:3100` |
| `docker compose up -d` says the container name `turtle` is already in use | An old turtle container is still there | Remove it with `docker rm -f turtle`, then start again |
| Changes don't show up in the browser (Windows) | Docker Desktop doesn't pass file changes from a Windows drive | Add `CHOKIDAR_USEPOLLING=true` to `.env`, then run `docker compose up -d` again |
| Your console login doesn't work in the app | App accounts and Fleetless accounts are separate | Sign in with the password you set when you accepted the invitation |

Load the message types:

```text
Fetch the message types of the robot turtle with
console_introspection_write, action fetch_types, then publish its
configuration again.
```

Allow the app's address:

```text
Add http://localhost:3000 to the allowed_origins of the app
turtle_control, keeping everything else in its registration slice as
it is.
```

## 📄 License

MIT, see [`LICENSE`](LICENSE). The turtle comes from
[turtle-sim](https://github.com/fleetless/turtle-sim) and the app from the
[app-starter](https://github.com/fleetless/app-starter). Everything else is
in the [Fleetless docs](https://docs.fleetless.dev).
