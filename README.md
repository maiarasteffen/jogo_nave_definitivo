# Jogo da Nave 


Esses códigos para o jogo foi desenvolvido segundo o livro: "Desenvolva Jogos com HTML Canvas e JavaScript", escrito por Éderson Cássio!

## Índice

- [Overview](#overview)
  - [O desafio](#o-desafio)
  - [Print do projeto](#print-do-projeto)
  - [Link](#link)
- [Meu processo](#meu-processo)
  - [Construído com](#construido-com)
  - [O que aprendi](#o-que-aprendi)
- [Autora](#autora)
- [Agradecimentos](#agradecimentos)

## Overview

### O desafio

O desafio constiste em:

- trabalhar com a tag canvas no HTML;
- desenvolver aplicações com javascript com o context;
- criar um painel de acordo com seu tamanho em y e x;
- criar movimentação com as imagens distribuídas;
- criar função para sons;
- criar funções para movimentação quando o usuário usar o teclado;
- criar uma função para alterar a imagem quando ocorre uma colisão;

### Link

- Código do desenvolvimento: [Jogo Nave](https://github.com/maiarasteffen/jogo_nave_definitivo)
- Front do projeto: [Front](https://tranquil-cupcake-b114ef.netlify.app/)

## Meu processo

### Construído com

- Canvas;
- Context;
- Prototype;
- Keydown e keyup;
- Spritesheet;

### O que aprendi

Aprendi muito sobre canvas no javascript e como desenvolver funções através de protótipos, além de criar funções para as teclas de setas, Enter e Espaço.

Código utilizado:

```html
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Jogo da Nave</title>
    <script src="animacao.js"></script>
    <script src="teclado.js"></script>
    <script src="colisor.js"></script>
    <script src="fundo.js"></script>
    <script src="nave.js"></script>
    <script src="ovni.js"></script>
    <script src="tiro.js"></script>
    <script src="spritesheet.js"></script>
    <script src="explosao.js"></script>
    <script src="painel.js"></script>
    <style>
        #link_jogar {
            display: none;
            color: yellow;
            background: url(img/botao-jogar.png);
            font-size: 20px;
            font-family: sans-serif;
            text-decoration: none;
            text-shadow: 2px 2px 5px black;
            position: absolute;
            left: 220px;
            top: 330px;
            width: 52px;
            height: 26px;
            padding: 23px 10px;
        }
    </style>
</head>
<body>
    <canvas id="canvas_animacao" width="500" height="500"></canvas>

    <a id="link_jogar" href="javascript: iniciarJogo()">Jogar</a>
    
    <script>
        var canvas = document.getElementById('canvas_animacao');
        var context = canvas.getContext('2d');

        var imagens, animacao, teclado, colisor, nave, criadorInimigos;
        var totalImagens = 0, carregadas = 0;
        var musicaAcao;
        
        carregarImagens();
        carregarMusicas()

        function carregarImagens() {
            imagens = {
                espaco: 'fundo-espaco.png',
                estrelas: 'fundo-estrelas.png',
                nuvens: 'fundo-nuvens.png',
                nave: 'nave-spritesheet.png',
                ovni: 'ovni.png',
                explosao: 'explosao.png'
            };

            for(var i in imagens) {
                var img = new Image();
                img.src = 'img/' + imagens[i];
                img.onload = carregando;
                totalImagens++;

                imagens[i] = img;
            }
        }
        function carregando() {
            context.save();

            context.drawImage(imagens.espaco, 0, 0, canvas.width, canvas.height);

            context.fillStyle = 'white';
            context.strokeStyle = 'black';
            context.font = '50px sans-serif';
            context.fillText("Carregando...", 100, 200);
            context.strokeText("Carregando...", 100, 200);

            carregadas++;
            var tamanhoTotal = 300;
            var tamanho = carregadas / totalImagens * tamanhoTotal;
            context.fillStyle = 'yellow';
            context.fillRect(100, 250, tamanho, 50);

            context.restore();

            if(carregadas == totalImagens) {
                iniciarObjetos();
                mostrarLinkJogar();
            }
        }

        function iniciarObjetos() {
            animacao = new Animacao(context);
            teclado = new Teclado(document);
            colisor = new Colisor();
            espaco = new Fundo(context, imagens.espaco);
            estrelas = new Fundo(context, imagens.estrelas);
            nuvens = new Fundo(context, imagens.nuvens);
            nave = new Nave(context, teclado, imagens.nave, imagens.explosao);
            painel = new Painel(context, nave);
                
            animacao.novoSprite(espaco);
            animacao.novoSprite(estrelas);
            animacao.novoSprite(nuvens);
            animacao.novoSprite(painel);
            animacao.novoSprite(nave);


            colisor.novoSprite(nave);
            animacao.novoProcessamento(colisor);

            configuracoesIniciais();
        }

        function configuracoesIniciais() {
            espaco.velocidade = 60;
            estrelas.velocidade = 150;
            nuvens.velocidade = 500;

            nave.posicionar();
            nave.velocidade = 200;

            criacaoInimigos();

            nave.acabaramVidas = function() {
                animacao.desligar();
                gameOver();
            }

            colisor.aoColidir = function(o1, o2) {
                if((o1 instanceof Tiro && o2 instanceof Ovni) || (o1 instanceof Ovni && o2 instanceof Tiro))
                    painel.pontuacao += 10;
            }
        }
        
        function criacaoInimigos() {
            criadorInimigos = {
                ultimoOvni: new Date().getTime(),

                processar: function() {
                    var agora = new Date().getTime();
                    var decorrido = agora - this.ultimoOvni;

                    if(decorrido > 1000) {
                        novoOvni();
                        this.ultimoOvni = agora;
                    }
                }
            };
            animacao.novoProcessamento(criadorInimigos);
        }
        function novoOvni() {
            var imgOvni = imagens.ovni;
            var ovni = new Ovni(context, imgOvni, imagens.explosao);

            ovni.velocidade = Math.floor(500 + Math.random() * (1000 - 500 + 1));

            ovni.x = Math.floor(Math.random() * (canvas.width - imgOvni.width + 1));

            ovni.y = - imgOvni.height;

            animacao.novoSprite(ovni);
            colisor.novoSprite(ovni);
        }
        function pausarJogo() {
         if (animacao.ligado) {
            animacao.desligar();
            ativarTiro(false);
            context.save();
            context.fillStyle = 'white';
            context.strokeStyle = 'black';
            context.font = '50px sans-serif';
            context.fillText("Pausado", 160, 200);
            context.strokeText("Pausado", 160, 200);
            context.restore();
        }
         else {
            criadorInimigos.ultimoOvni = new Date().getTime();
            animacao.ligar();
            ativarTiro(true);
        }
    }
        function ativarTiro(ativar) {
            if(ativar) {
                teclado.disparou(ESPACO, function() {
                    nave.atirar();
                });
            }
            else {
                teclado.disparou(ESPACO, null);
            }
        }
        function carregarMusicas() {
            musicaAcao = new Audio();
            musicaAcao.src = 'snd/musica-acao.mp3';
            musicaAcao.volume = 0.8;
            musicaAcao.loop = true;
            musicaAcao.load();
        }
        function mostrarLinkJogar() {
            document.getElementById('link_jogar').style.display = 'block';
        }
        function iniciarJogo() {
            criadorInimigos.ultimoOvni = new Date().getTime();

            ativarTiro(true);

            teclado.disparou(ENTER, pausarJogo);

            document.getElementById('link_jogar').style.display = 'none';
            musicaAcao.play();
            animacao.ligar();
        }
        function gameOver() {
            ativarTiro(false);

            teclado.disparou(ENTER, null);

            musicaAcao.pause();
            musicaAcao.currentTime = 0.0;

            context.drawImage(imagens.espaco, 0, 0, canvas.width, canvas.height);

            context.save();
            context.fillStyle = 'white';
            context.strokeStyle = 'black';
            context.font = '70px sans-serif';
            context.fillText("GAME OVER", 40, 200);
            context.strokeText("GAME OVER", 40, 200);
            context.restore();

            mostrarLinkJogar();

            nave.vidasExtras = 3;
            nave.posicionar();
            animacao.novoSprite(nave);
            colisor.novoSprite(nave);

            removerInimigos();
        }
        function removerInimigos() {
            for(var i in animacao.sprites) {
                if(animacao.sprites[i] instanceof Ovni)
                    animacao.excluirSprite(animacao.sprites[i]);
            }
        }
    </script>
</body>
</html>
```
## Author

- Linkedin - [Maiara Steffen](https://www.linkedin.com/in/maiara-steffen/)
- Frontend Mentor - [@maiarasteffen](https://www.frontendmentor.io/profile/maiarasteffen)
- Instagram - [@maiara_steffen](https://www.instagram.com/maiara_steffen/)
- GitHub - [@maiarasteffen](https://github.com/maiarasteffen/)

## Agradecimentos

Primeiro quero agradecer muito a Deus por sempre estar me dando oportunidades de me desenvolver cada vez mais na carreira de programadora, aos professores e alunos da faculdade que fiz, pois um compartilhava com o outro conhecimentos e materias para que pudessemos sempre estar nos aperfeiçoando!
