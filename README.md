from _typeshed import Self
from flask import Flask, Response, Request
from flask_sqlalchemy import SQLAlchemy
import _mysql_connector
import json

from werkzeug.wrappers import request

app = Flask(__name__)
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = True
app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql://root:@localhost/base'

db = SQLAlchemy(app)

#define um modelo 
class Filme(db.Model):
    id = db.Column(db.Integer, primary_key = True)
    nome = db.Column(db.String(50))

#tranforma em json
def to_json(self):
    return{"id": self.id, "nome": self.nome}

#cadastrar um novo filme
@app.route("/filmes", methods=["POST"])
def cadastrar():
    body = request.get_json()


    try:
        filme = Filme(nome=body["nome"],id = body["id"])
        db.session.add(filme)
        db.session.commit()
        return gera_response(201, "filme", filme.to_json(), "Cadastrado com sucesso")

    except Exception as e:
        print(e)
        return gera_response(400,"usuario",{}, "Erro ao cadastrar")

#retornar todos os filmes
@app.route("/filmes", methods=["GET"])
def retorna_filmes():
    filmes_objetos = Filme.query.all()
    filmes_json = [filme.to_json()for filme in filmes_objetos]

    return gera_response(200,"filmes", filmes_json)

#retornar pelo id
@app.route("/filmes/<id>")
def retorna_filmes(id):
    filme_objeto = Filme.query.filter_by(id=Self.id).first()
    filme_json = filme_objeto.to_json()

    return gera_response(200, "filme", filme_json)

def gera_response(status, nome_do_conteudo, conteudo, mensagem=False):
    body = {}
    body[nome_do_conteudo] = conteudo

    if(mensagem):
        body["mensagem"] = mensagem

    return Response(json.dumps(body), status=status, mimetype="application/json")


    app.run()
