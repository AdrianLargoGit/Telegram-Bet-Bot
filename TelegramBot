import logging
import requests
from datetime import datetime
from telegram import Update
from telegram.ext import Application, CommandHandler, CallbackContext

# Claves API
API_KEY = ""  # Reemplaza con tu clave API de Football-Data.org
TELEGRAM_BOT_TOKEN = ""  # Reemplaza con tu token de Telegram

# Configuración de logging
logging.basicConfig(format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
                    level=logging.INFO)
logger = logging.getLogger(__name__)

# Función para obtener los partidos de hoy desde la API de Football-Data.org
def get_matches():
    today = datetime.now().strftime('%Y-%m-%d')
    url = 'https://api.football-data.org/v4/matches'
    headers = {
        'X-Auth-Token': API_KEY
    }
    params = {
        'dateFrom': today,
        'dateTo': today
    }

    response = requests.get(url, headers=headers, params=params)

    if response.status_code != 200:
        logger.error(f"Error en la solicitud a la API. Código de estado: {response.status_code}")
        return []

    try:
        data = response.json()
        if 'matches' in data:
            return data['matches']
        else:
            logger.error("No se encontró la clave 'matches' en la respuesta de la API.")
            return []

    except ValueError:
        logger.error("Error al convertir la respuesta a JSON.")
        return []

# Función para obtener las estadísticas de los equipos
def get_team_stats(team_id):
    url = f'https://api.football-data.org/v4/teams/{team_id}'
    headers = {
        'X-Auth-Token': API_KEY
    }

    response = requests.get(url, headers=headers)

    if response.status_code != 200:
        logger.error(f"Error al obtener estadísticas del equipo {team_id}. Código de estado: {response.status_code}")
        return {}

    try:
        team_data = response.json()
        return team_data.get('statistics', {})
    except ValueError:
        logger.error("Error al convertir la respuesta a JSON.")
        return {}

# Función para generar las predicciones basadas en estadísticas
def generate_predictions(local_team, away_team, local_stats, away_stats):
    local_goals_avg = local_stats.get('average', {}).get('goals', 0)
    away_goals_avg = away_stats.get('average', {}).get('goals', 0)

    local_yellow_cards = local_stats.get('average', {}).get('yellowCards', 0)
    away_yellow_cards = away_stats.get('average', {}).get('yellowCards', 0)

    local_fouls = local_stats.get('average', {}).get('fouls', 0)
    away_fouls = away_stats.get('average', {}).get('fouls', 0)

    local_corners = local_stats.get('average', {}).get('corners', 0)
    away_corners = away_stats.get('average', {}).get('corners', 0)

    local_shots = local_stats.get('average', {}).get('shots', 0)
    away_shots = away_stats.get('average', {}).get('shots', 0)

    local_on_target = local_stats.get('average', {}).get('onTarget', 0)
    away_on_target = away_stats.get('average', {}).get('onTarget', 0)

    num_goals_local = round(local_goals_avg)
    num_goals_away = round(away_goals_avg)

    prediction = {
        'score': f'{local_team} {num_goals_local}-{num_goals_away} {away_team}',
        'yellow_cards': f'{local_team} {local_yellow_cards} - {away_team} {away_yellow_cards}',
        'red_cards': f'{local_team} 0 - {away_team} 0',
        'fouls': f'{local_team} {local_fouls} - {away_team} {away_fouls}',
        'corners': f'{local_team} {local_corners} - {away_team} {away_corners}',
        'shots': f'{local_team} {local_shots} - {away_team} {away_shots}',
        'shots_on_target': f'{local_team} {local_on_target} - {away_team} {away_on_target}'
    }

    return prediction

# Comando /start para iniciar el bot
async def start(update: Update, context: CallbackContext):
    await update.message.reply_text('¡Hola! Soy WizardOfBets, tu bot de pronósticos de fútbol. Estaré prediciendo los partidos del día.')

    matches = get_matches()

    if not matches:
        await update.message.reply_text('No se encontraron partidos para hoy.')
        return

    # Enviar pronósticos para cada partido
    for match in matches:
        local_team = match['homeTeam']['name'] if 'homeTeam' in match else 'Equipo Local'
        away_team = match['awayTeam']['name'] if 'awayTeam' in match else 'Equipo Visitante'
        match_time = datetime.strptime(match['utcDate'], '%Y-%m-%dT%H:%M:%SZ').strftime('%H:%M')

        local_team_id = match['homeTeam']['id']
        away_team_id = match['awayTeam']['id']

        local_stats = get_team_stats(local_team_id)
        away_stats = get_team_stats(away_team_id)

        prediction = generate_predictions(local_team, away_team, local_stats, away_stats)

        message = (
            f"{local_team} vs {away_team}\n"
            f"Hora: {match_time}\n"
            f"Pronóstico:\n"
            f"Marcador estimado: {prediction['score']}\n"
            f"Tarjetas Amarillas: {prediction['yellow_cards']}\n"
            f"Tarjetas Rojas: {prediction['red_cards']}\n"
            f"Faltas cometidas: {prediction['fouls']}\n"
            f"Corners: {prediction['corners']}\n"
            f"Remates Totales: {prediction['shots']}\n"
            f"Remates a puerta: {prediction['shots_on_target']}\n"
        )

        await update.message.reply_text(message)

# Función para manejar errores
async def error(update: Update, context: CallbackContext):
    logger.warning(f"Update {update} caused error {context.error}")

# Main function para iniciar el bot
def main():
    application = Application.builder().token(TELEGRAM_BOT_TOKEN).build()
    application.add_handler(CommandHandler("start", start))
    application.add_error_handler(error)
    application.run_polling()

if __name__ == '__main__':
    main()
