import keep_alive
import os
import discord
import random
import json
from discord.ext import commands
from dislash import InteractionClient

client = commands.Bot(command_prefix="-")
client.remove_command("help")
slash = InteractionClient(client)
test_guilds = [12345, 98765]

@client.event
async def on_ready():
  print('We have logged in as {0.user}'.format(client))

#------------------------------------------

@slash.command(description="Shows your balance", aliases=["bal"])
async def balance(ctx):
  await open_account(ctx.author)
  user = ctx.author
  users = await get_bank_data()

  wallet_amt = users[str(user.id)]["wallet"]
  bank_amt = users[str(user.id)]["bank"]

  em = discord.Embed(title = f"{ctx.author.name}´s balance",color = discord.Color.red())
  em.add_field(name = "Wallet balance",value = wallet_amt)
  em.add_field(name = "Bank balance",value = bank_amt)
  await ctx.send(embed = em)

@slash.command(description="Get an amount between 1 - 100 from a random stranger")
@commands.cooldown(1,60,commands.BucketType.user)
async def beg(ctx):
  await open_account(ctx.author)

  user = ctx.author

  users = await get_bank_data()

  earnings = random.randrange(0, 101)

  await ctx.send(f"Someone gave you {earnings} coins!!")

  users[str(user.id)]["wallet"] += earnings

  with open("mainbank.json","w") as f: 
    json.dump(users,f)

async def open_account(user):
  
  users = await get_bank_data()

  if str(user.id) in users:
    return False
  else: 
    users[str(user.id)] = {}
    users[str(user.id)]["wallet"] = 0
    users[str(user.id)]["bank"] = 0
  
  with open("mainbank.json","w") as f: 
    json.dump(users,f)
  return True

async def get_bank_data():
  with open("mainbank.json","r") as f: 
    users = json.load(f)

  return users 

async def update_bank(user,change = 0,mode = "wallet"):
  users = await get_bank_data()

  users[str(user.id)][mode] += change

  with open("mainbank.json","w") as f: 
    json.dump(users,f)
  balance = [users[str(user.id)]["wallet"],users[str(user.id)]["bank"]]
  return balance

@client.event
async def on_command_error(ctx, error):
  if isinstance(error, commands.CommandOnCooldown):
    msg = "People aren't made out of money, try again in {:.2f}s".format(error.retry_after)
    await ctx.send(msg)

#------------------------------------------

@slash.command(description="Roll the 6-dice")
async def dice6(ctx):
  n = random.randrange(1, 6)

  await ctx.send(f"🎲 You rolled {n}!")

@slash.command(description="Roll the 10-dice")
async def dice10(ctx):
  n = random.randrange(1, 10)
  
  await ctx.send(f"🎲 You rolled {n}!")

#------------------------------------------

@slash.command(description="Opens up the help menu")
async def help(ctx):

  help = discord.Embed(title = f"{ctx.author.name} used /help",color = discord.Color.red())
  help.add_field(name = "help",value = "Opens up this screen")
  help.add_field(name = "helpfun",value = "Opens up the help fun screen")
  help.add_field(name = "helpcurrency",value = "Opens up the help currency screen")
  help.add_field(name = "settings",value = "Opens up the settings screen")
  await ctx.send(embed = help)

@slash.command(description="Opens up the settings screen")
async def settings(ctx):

  settings = discord.Embed(title = f"{ctx.author.name} used /settings",color = discord.Color.red())
  settings.add_field(name = "prefix",value = "Changes the prefix (Work in Progress)")
  await ctx.send(embed = settings)

@slash.command(description="Opens up the help fun screen")
async def helpfun(ctx):

  fun = discord.Embed(title = f"{ctx.author.name} used /helpfun",color = discord.Color.red())
  fun.add_field(name = "dice6",value = "Random number between 1 and 6")
  fun.add_field(name = "dice10",value = "Random number between 1 and 10")
  await ctx.send(embed = fun)

@slash.command(description="Opens up the help currency screen")
async def helpcurrency(ctx):

  currency = discord.Embed(title = f"{ctx.author.name} used /helpcurrency",color = discord.Color.red())
  currency.add_field(name = "balance (bal)",value = "Shows your balance")
  currency.add_field(name = "beg",value = "Get an amount between 1 - 100 from a random stranger")
  await ctx.send(embed = currency)

#------------------------------------------

keep_alive.keep_alive()

client.run(os.environ['TOKEN'])
