from pyrogram import Client, filters
from pytgcalls import PyTgCalls
from pytgcalls.types.input_stream import AudioPiped
import yt_dlp
import os

api_id = int(os.getenv("API_ID"))
api_hash = os.getenv("API_HASH")
bot_token = os.getenv("BOT_TOKEN")

app = Client("musicbot", api_id=api_id, api_hash=api_hash, bot_token=bot_token)
calls = PyTgCalls(app)

@app.on_message(filters.command("play"))
async def play(_, message):
    if len(message.command) < 2:
        return await message.reply("Ketik /play judul lagu")

    query = " ".join(message.command[1:])
    await message.reply(f"Mencari: {query}")

    ydl_opts = {
        "format": "bestaudio",
        "outtmpl": "song.mp3"
    }

    with yt_dlp.YoutubeDL(ydl_opts) as ydl:
        ydl.download([f"ytsearch:{query}"])

    await calls.join_group_call(
        message.chat.id,
        AudioPiped("song.mp3")
    )

    await message.reply("🎶 Lagu diputar!")

@app.on_message(filters.command("stop"))
async def stop(_, message):
    await calls.leave_group_call(message.chat.id)
    await message.reply("⏹️ Musik berhenti")

app.start()
calls.start()
app.idle()