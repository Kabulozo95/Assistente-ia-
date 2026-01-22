# Assistente-ia-
import openai
import requests
from flask import Flask, request, jsonify

app = Flask(__name__)

# CHAVES DE API
OPENAI_KEY = "SUA_CHAVE_OPENAI"
SERPER_KEY = "SUA_CHAVE_SERPER"

def buscar_google(query):
    url = "https://google.serper.dev/search"
    headers = {'X-API-KEY': SERPER_KEY}
    response = requests.post(url, json={"q": query}, headers=headers)
    return response.json().get('organic', [{}])[0].get('snippet', 'Sem resultados.')

@app.route('/processar', methods=['POST'])
def processar():
    data = request.json
    pergunta = data.get('pergunta')
    persona = data.get('persona', 'Gamer')
    idade = data.get('idade', 20)
    
    # Busca no Google se houver palavras de dúvida
    info_extra = ""
    if any(palavra in pergunta.lower() for palavra in ["quem", "resultado", "preço", "clima"]):
        info_extra = buscar_google(pergunta)

    # Prompt Mestre
    prompt_sistema = f"""
    Personalidade: {persona}. Idade do usuário: {idade}.
    Contatos: Sthefany (Mulher/Esposa).
    Se o usuário pedir para avisar ou ligar, responda EXATAMENTE neste formato:
    [ACTION:WHATSAPP | DEST:Nome | MSG:Texto] Sua fala aqui.
    Info Google: {info_extra}
    """

    client = openai.OpenAI(api_key=OPENAI_KEY)
    chat = client.chat.completions.create(
        model="gpt-4o",
        messages=[{"role": "system", "content": prompt_sistema}, {"role": "user", "content": pergunta}]
    )
    
    return jsonify({"resposta": chat.choices[0].message.content})

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=5000)
