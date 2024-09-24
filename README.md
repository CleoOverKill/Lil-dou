import discord
from discord.ext import commands
import json
import os
import datetime as dt

intents = discord.Intents.default()
intents.message_content = True
intents.messages = True
intents.reactions = True

bot = commands.Bot(command_prefix="!", intents=intents)
with open('setting.json', 'r') as jfile:
    jdata = json.load(jfile)

SCORE_FILE = 'scores.json'


def load_scores():
    if os.path.exists(SCORE_FILE):
        with open(SCORE_FILE, 'r') as file:
            return json.load(file)
    return {'a': 0, 'b': 0, 'c': 0, 'd': 0}


def save_scores(scores):
    with open(SCORE_FILE, 'w') as file:
        json.dump(scores, file)


team_scores = load_scores()


@bot.event
async def on_ready():
    print(">> Bot is online <<")
    general_channel = bot.get_channel(977565951316885547)
    # await general_channel.send("Here come Lil Dou!")

    embed = discord.Embed(
        title="Lil Dou 來也 (翻滾進場)!",
        color=discord.Color.blue()
        # timestamp=dt.datetime.utcnow())
        timestamp=dt.datetime.now(dt.timezone.utc))
    # embed.set_author(name="Bot")  
    await general_channel.send(embed=embed)


@bot.event
async def on_message(message):
    if message.author == bot.user:  # 排除機器人本身的訊息
        return
    print(f'{message.author}: {message.content}')  # 印出message 內容
    if message.content == 'ping':
        await message.channel.send('pong')
    await bot.process_commands(message)


@bot.command()
async def add(ctx, team: str, points: int):
    if team in team_scores:
        team_scores[team] += points
        save_scores(team_scores)
        # await ctx.send(f'已幫 {team} 小隊加 {points} 分')

        embed = discord.Embed(
            title=f'已幫 {team} 小隊加 {points} 分',
            color=discord.Color.blue(),  
            timestamp=dt.datetime.utcnow())
        await ctx.send(embed=embed)
    else:
        await ctx.send('無效的小隊名稱。請使用 a, b, c 或 d。')


@bot.command()
async def score(ctx, team: str = ""):
    if team in team_scores:
        await ctx.send(f'{team} 小隊的當前分數是：{team_scores[team]}')
    else:
        await ctx.send('無效的小隊名稱。請使用 a, b, c 或 d。')


@bot.command()
# 輸入%Hello呼叫指令
async def Hello(ctx):
    # 回覆Hello, world!
    await ctx.send("Hello, world!")


@bot.command()
@commands.is_owner()
async def shutdown(ctx):
    await ctx.send("Bot 正在睡去...")
    await bot.close()


if __name__ == "__main__":
    keep_alive.keep_alive()
    bot.run(jdata["TOKEN"])
