A example for a daily link switch would be: 
import discord
import random
import schedule
import time
import asyncio

DISCORD_TOKEN = 'Your Discord Token'
current_link = 'Current Link' 

def generate_party_link():
    random_number = ''.join([str(random.randint(0, 9)) for _ in range(10)])
    party_link = f'https://wormax.io/?party={random_number}'
    return party_link

def create_and_upload_link():
    global current_link
    try:
        party_link = generate_party_link()
        current_link = party_link
        print(f'New Wormax Party Link: {current_link}')
    except Exception as e:
        print(f'Failed to create link: {e}')

# Discord Bot Setup
intents = discord.Intents.default()  # Standard-Intents initialisieren
client = discord.Client(intents=intents)

@client.event
async def on_ready():
    print(f'We have logged in as {client.user}')
    create_and_upload_link()  # Generate the initial link when bot starts
    schedule_days_task()  # Start the weekly task

@client.event
async def on_message(message):
    if message.author == client.user:
        return

    if message.content.startswith('!link'):
        if current_link:
            await message.channel.send(f'Current Wormax Party Link: {current_link}')
            # Log the user who requested the link
            print(f'Link requested by: {message.author.name}#{message.author.discriminator}')
        else:
            await message.channel.send('No link available at the moment.')
            # Log the user who requested the link
            print(f'Link requested by: {message.author.name}#{message.author.discriminator} but no link available.')

# Daily switch
def schedule_days_task():
    schedule.every().days.do(create_and_upload_link)

    async def run_scheduler():
        while True:
            schedule.run_pending()
            await asyncio.sleep(1)

    client.loop.create_task(run_scheduler())

client.run(DISCORD_TOKEN)
