- 👋 Hi, I’m @Robis911
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...

<!---
Robis911/Robis911 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.

import requests
import json
import re
import time
import base64

player_id = "YOUR_PLAYER_ID"
level_id = "YOUR_LEVEL_ID"

session = requests.Session()

headers = {
    'User-Agent': 'King-com-user-mobile-api/internal-testing-20130905-1022 exp=0 session=4703410213112111500',
    'X-Supersonic-App-Id': 'com.king.candycrushsaga',
    'X-Supersonic-App-Version': '1.58.0.3',
    'X-Supersonic-User-Id': player_id,
    'Content-Type': 'application/json'
}

def get_level():
    url = f'https://ccs-api-useast1.king.com:443/levels/{level_id}?exclude_episodes=false'
    response = session.get(url, headers=headers)
    return response.json()

def send_move(move):
    url = 'https://ccs-api-useast1.king.com:443/v3/sessions/active'
    data = json.dumps({
        'requests': [{
            'type': 'move',
            'move': move
        }]
    })
    response = session.post(url, data=data, headers=headers)
    return response.json()

def crack_level():
    level = get_level()
    columns = level['columns']
    rows = level['rows']
    max_score = level['max_score']
    max_score_achieved = 0

    board = []
    for row in range(rows):
        board.append([])
        for column in range(columns):
            board[row].append(' ')

    moves = []
    for row in range(rows):
        for column in range(columns):
            cell = level['cells'][row][column]
            if 'element' in cell:
                board[row][column] = cell['element']
                if cell['is_mine']:
                    moves.append((row, column))

    while max_score_achieved < max_score:
        for move in moves:
            row, column = move
            session.headers.update({'X-Supersonic-Board-Id': level['board_id']})
            response = send_move([row, column])
            new_board = response['sessions'][0]['board']
            for i in range(rows):
                for j in range(columns):
                    if board[i][j] != new_board[i][j]:
                        board = new_board
                        max_score_achieved = response['sessions'][0]['score']
                        break
        time.sleep(0.1)

crack_level()
