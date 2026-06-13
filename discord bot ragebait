import discord
from discord.ext import commands
import re
import random
import asyncio
import json
import os
from aiohttp import web

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# CONFIGURATION
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TOKEN = os.getenv("DISCORD_TOKEN", "TON_TOKEN_ICI")
OWNER_ID = 123456789012345678  # ← TON ID Discord (pour contrôler le bot en DM)
CONFIG_FILE = "bot_config.json"

# Config par défaut
DEFAULT_CONFIG = {
    "channel_id": 0,
    "target_id": 939940429485473872,
    "mode": "Bébé",
    "frequency": 100,
    "enabled": True,
    "reply": False,
    "react": False,
    "reaction": "😂",
    "delay_min": 0.5,
    "delay_max": 2.0,
}

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# MODES DE TROLL
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

def mode_bebe(text):
    replacements = [
        (r"\bje\b", "ze"), (r"\bJe\b", "Ze"),
        (r"\bj'", "z'"), (r"\bJ'", "Z'"),
        (r"ch", "ts"), (r"Ch", "Ts"),
        (r"tr", "tw"), (r"Tr", "Tw"),
        (r"cr", "cw"), (r"Cr", "Cw"),
        (r"gr", "gw"), (r"Gr", "Gw"),
        (r"pr", "pw"), (r"Pr", "Pw"),
        (r"br", "bw"), (r"Br", "Bw"),
        (r"fr", "fw"), (r"Fr", "Fw"),
        (r"dr", "dw"), (r"Dr", "Dw"),
        (r"r", "w"), (r"R", "W"),
        (r"l", "w"), (r"L", "W"),
    ]
    result = text
    for pattern, replacement in replacements:
        result = re.sub(pattern, replacement, result)
    return result

def mode_inverse(text):
    return text[::-1]

def mode_majuscules(text):
    return text.upper()

def mode_alternes(text):
    return ''.join(c.upper() if i % 2 == 0 else c.lower() for i, c in enumerate(text))

def mode_spaces(text):
    return ' '.join(list(text))

def mode_uwu(text):
    replacements = [(r"r", "w"), (r"R", "W"), (r"l", "w"), (r"L", "W")]
    result = text
    for pattern, replacement in replacements:
        result = re.sub(pattern, replacement, result)
    return result + random.choice([" uwu", " owo", " >w<", " :3"])

def mode_sarcastique(text):
    return ''.join(c.upper() if i % 2 == 0 else c.lower() for i, c in enumerate(text))

def mode_echo(text):
    words = text.split()
    if words:
        last = words[-1].lower()
        return f"{text} {last}... {last}..."
    return text

def mode_pirate(text):
    result = re.sub(r"\boui\b", "aye", text, flags=re.IGNORECASE)
    result = re.sub(r"\bnon\b", "nan mille sabords", result, flags=re.IGNORECASE)
    result = re.sub(r"\bsalut\b", "yarr", result, flags=re.IGNORECASE)
    if not result.endswith(("!", "?")):
        result += ", arrr"
    return result

def mode_robot(text):
    text = text.upper()
    text = re.sub(r'\s+', '.', text)
    return f"BLEEP.BLOOP.{text}.FIN.TRANSMISSION"

def mode_dramatique(text):
    intros = ["*entre en scène*", "*ajuste son monocle*", "*inspiration dramatique*"]
    outros = ["*tonnerre au loin*", "*une larme coule*", "*rideau*"]
    return f"{random.choice(intros)} {text.upper()} {random.choice(outros)}"

def mode_censure(text):
    vowels = "aeiouyAEIOUY"
    return ''.join('*' if c in vowels else c for c in text)

def mode_copie(text):
    return text

def mode_quotes(text):
    return f'"{text}"'

MODES = {
    "bebe": mode_bebe,
    "inverse": mode_inverse,
    "majuscules": mode_majuscules,
    "alternes": mode_alternes,
    "spaces": mode_spaces,
    "uwu": mode_uwu,
    "sarcastique": mode_sarcastique,
    "echo": mode_echo,
    "pirate": mode_pirate,
    "robot": mode_robot,
    "dramatique": mode_dramatique,
    "censure": mode_censure,
    "copie": mode_copie,
    "quotes": mode_quotes,
}

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# GESTION CONFIG
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

def load_config():
    if os.path.exists(CONFIG_FILE):
        with open(CONFIG_FILE, "r") as f:
            saved = json.load(f)
            return {**DEFAULT_CONFIG, **saved}
    return DEFAULT_CONFIG.copy()

def save_config(config):
    with open(CONFIG_FILE, "w") as f:
        json.dump(config, f, indent=2)

config = load_config()

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# BOT DISCORD
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

intents = discord.Intents.default()
intents.message_content = True
bot = commands.Bot(command_prefix="!", intents=intents)

def is_owner(ctx):
    return ctx.author.id == OWNER_ID

# ━━━ COMMANDES DM ━━━

@bot.command(name="help", aliases=["h", "aide"])
async def help_cmd(ctx):
    if not is_owner(ctx):
        return
    
    help_text = """
**🎭 COMMANDES DU TROLL BOT**

**⚙️ Configuration:**
`!status` — Voir la config actuelle
`!channel <id>` — Changer le salon
`!cible <id>` — Changer la cible (0 = tout le monde)
`!mode <nom>` — Changer le mode de troll
`!modes` — Liste des modes disponibles
`!freq <1-100>` — Changer la fréquence (%)
`!delay <min> <max>` — Délai de réponse (secondes)
`!reply on/off` — Répondre au message ou non
`!react on/off <emoji>` — Ajouter une réaction

**🎮 Contrôle:**
`!on` — Activer le bot
`!off` — Désactiver le bot
`!say <message>` — Envoyer un message (brut)
`!troll <message>` — Envoyer un message (avec le mode actif)

**💾 Sauvegarde:**
`!save` — Sauvegarder la config
"""
    await ctx.send(help_text)

@bot.command(name="status", aliases=["s", "config"])
async def status_cmd(ctx):
    if not is_owner(ctx):
        return
    
    channel = bot.get_channel(config["channel_id"])
    channel_name = f"#{channel.name}" if channel else "Non trouvé"
    
    status = f"""
**📊 STATUS DU BOT**

🔘 État: {"🟢 Activé" if config["enabled"] else "🔴 Désactivé"}
📡 Salon: `{config["channel_id"]}` ({channel_name})
🎯 Cible: `{config["target_id"]}` {"(tout le monde)" if config["target_id"] == 0 else ""}
🎭 Mode: `{config["mode"]}`
🎲 Fréquence: `{config["frequency"]}%`
⏱️ Délai: `{config["delay_min"]}s - {config["delay_max"]}s`
↩️ Reply: `{"Oui" if config["reply"] else "Non"}`
😂 Réaction: `{"Oui " + config["reaction"] if config["react"] else "Non"}`
"""
    await ctx.send(status)

@bot.command(name="channel", aliases=["salon"])
async def channel_cmd(ctx, channel_id: int):
    if not is_owner(ctx):
        return
    config["channel_id"] = channel_id
    channel = bot.get_channel(channel_id)
    if channel:
        await ctx.send(f"✅ Salon changé: `{channel_id}` (#{channel.name})")
    else:
        await ctx.send(f"⚠️ Salon `{channel_id}` défini mais introuvable (vérifie l'ID)")

@bot.command(name="cible", aliases=["target", "c"])
async def cible_cmd(ctx, target_id: int):
    if not is_owner(ctx):
        return
    config["target_id"] = target_id
    if target_id == 0:
        await ctx.send("✅ Cible: **Tout le monde**")
    else:
        await ctx.send(f"✅ Cible changée: `{target_id}`")

@bot.command(name="mode", aliases=["m"])
async def mode_cmd(ctx, mode_name: str):
    if not is_owner(ctx):
        return
    mode_lower = mode_name.lower()
    if mode_lower in MODES:
        config["mode"] = mode_lower
        # Preview
        preview = MODES[mode_lower]("Je vais travailler aujourd'hui")
        await ctx.send(f"✅ Mode changé: `{mode_lower}`\n📝 Exemple: *{preview}*")
    else:
        await ctx.send(f"❌ Mode inconnu. Utilise `!modes` pour voir la liste.")

@bot.command(name="modes", aliases=["listmodes"])
async def modes_cmd(ctx):
    if not is_owner(ctx):
        return
    
    example = "Salut je travaille"
    text = "**🎭 MODES DISPONIBLES:**\n\n"
    for name, func in MODES.items():
        text += f"`{name}` → *{func(example)}*\n"
    await ctx.send(text)

@bot.command(name="freq", aliases=["f", "frequence"])
async def freq_cmd(ctx, freq: int):
    if not is_owner(ctx):
        return
    freq = max(1, min(100, freq))
    config["frequency"] = freq
    await ctx.send(f"✅ Fréquence: `{freq}%`")

@bot.command(name="delay", aliases=["delai"])
async def delay_cmd(ctx, min_delay: float, max_delay: float = None):
    if not is_owner(ctx):
        return
    if max_delay is None:
        max_delay = min_delay
    config["delay_min"] = min_delay
    config["delay_max"] = max_delay
    await ctx.send(f"✅ Délai: `{min_delay}s` à `{max_delay}s`")

@bot.command(name="reply")
async def reply_cmd(ctx, state: str):
    if not is_owner(ctx):
        return
    config["reply"] = state.lower() in ["on", "oui", "yes", "true", "1"]
    await ctx.send(f"✅ Reply: `{'Oui' if config['reply'] else 'Non'}`")

@bot.command(name="react")
async def react_cmd(ctx, state: str, emoji: str = "😂"):
    if not is_owner(ctx):
        return
    config["react"] = state.lower() in ["on", "oui", "yes", "true", "1"]
    config["reaction"] = emoji
    await ctx.send(f"✅ Réaction: `{'Oui ' + emoji if config['react'] else 'Non'}`")

@bot.command(name="on", aliases=["start", "activer"])
async def on_cmd(ctx):
    if not is_owner(ctx):
        return
    config["enabled"] = True
    await ctx.send("✅ Bot **activé** ! 🟢")

@bot.command(name="off", aliases=["stop", "desactiver"])
async def off_cmd(ctx):
    if not is_owner(ctx):
        return
    config["enabled"] = False
    await ctx.send("✅ Bot **désactivé** ! 🔴")

@bot.command(name="say", aliases=["dire", "send"])
async def say_cmd(ctx, *, message: str):
    if not is_owner(ctx):
        return
    channel = bot.get_channel(config["channel_id"])
    if channel:
        await channel.send(message)
        await ctx.send(f"✅ Message envoyé !")
    else:
        await ctx.send("❌ Salon introuvable")

@bot.command(name="troll")
async def troll_cmd(ctx, *, message: str):
    if not is_owner(ctx):
        return
    channel = bot.get_channel(config["channel_id"])
    if channel:
        mode_func = MODES.get(config["mode"], mode_copie)
        transformed = mode_func(message)
        await channel.send(transformed)
        await ctx.send(f"✅ Envoyé: *{transformed}*")
    else:
        await ctx.send("❌ Salon introuvable")

@bot.command(name="save", aliases=["sauvegarder"])
async def save_cmd(ctx):
    if not is_owner(ctx):
        return
    save_config(config)
    await ctx.send("✅ Configuration sauvegardée !")

# ━━━ ÉVÉNEMENTS ━━━

@bot.event
async def on_ready():
    print(f"✅ {bot.user} connecté !")
    print(f"📡 Salon: {config['channel_id']}")
    print(f"🎯 Cible: {config['target_id']}")
    print(f"🎭 Mode: {config['mode']}")
    print(f"👤 Owner: {OWNER_ID}")
    print("─" * 40)
    print("Envoie !help en DM au bot pour les commandes")

@bot.event
async def on_message(message):
    # Traiter les commandes d'abord
    await bot.process_commands(message)
    
    # Ignorer si c'est une commande ou le bot lui-même
    if message.author == bot.user:
        return
    if message.content.startswith("!"):
        return
    
    # Ignorer si désactivé
    if not config["enabled"]:
        return
    
    # Vérifier le salon
    if message.channel.id != config["channel_id"]:
        return
    
    # Vérifier la cible
    if config["target_id"] != 0 and message.author.id != config["target_id"]:
        return
    
    # Ignorer messages vides
    if not message.content.strip():
        return
    
    # Fréquence
    if random.randint(1, 100) > config["frequency"]:
        return
    
    print(f"[CIBLE] {message.author.name}: {message.content}")
    
    # Délai
    delay = random.uniform(config["delay_min"], config["delay_max"])
    await asyncio.sleep(delay)
    
    # Transformer
    mode_func = MODES.get(config["mode"], mode_copie)
    transformed = mode_func(message.content)
    
    # Envoyer
    if config["reply"]:
        await message.reply(transformed, mention_author=False)
    else:
        await message.channel.send(transformed)
    
    print(f"[BOT] {transformed}")
    
    # Réaction
    if config["react"] and config["reaction"]:
        try:
            await message.add_reaction(config["reaction"])
        except:
            pass

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# SERVEUR WEB (pour UptimeRobot)
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

async def handle_ping(request):
    return web.Response(text="OK")

async def run_webserver():
    app = web.Application()
    app.router.add_get("/", handle_ping)
    app.router.add_get("/ping", handle_ping)
    
    port = int(os.getenv("PORT", 8080))
    runner = web.AppRunner(app)
    await runner.setup()
    site = web.TCPSite(runner, "0.0.0.0", port)
    await site.start()
    print(f"🌐 Serveur web démarré sur le port {port}")

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# LANCEMENT
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

async def main():
    await run_webserver()
    await bot.start(TOKEN)

if __name__ == "__main__":
    asyncio.run(main())
