 <a href="#"><img src="https://komarev.com/ghpvc/?username=tskbrasil&style=for-the-badge&label=Views:&color=ff69b4"/></a>
# Cheat CMSP by iUnknown - Down - caiu

Um script de automação para o sistema de gestão de aprendizagem CMSP, que facilita o envio de respostas de atividades e lições.

## Descrição

O **Cheat CMSP** é um UserScript desenvolvido para automatizar o preenchimento de respostas em lições no CMSP. Ele transforma respostas recebidas do servidor e as envia automaticamente, permitindo que os usuários concluam suas atividades de forma mais rápida e eficiente.

## Funcionalidades

- **Automação de Respostas:** Envia automaticamente as respostas das lições.
- **Notificação de Conclusão:** Exibe uma notificação "Atividade Concluída" após o envio das respostas.
- **Alteração de Velocidade:** Permite que os usuários ajustem o tempo de atraso entre as ações.
- **Interface Amigável:** Um overlay que fornece informações sobre o uso do script e um link para o Discord.

## Instalação

Para usar o **Cheat CMSP**, siga as etapas abaixo:

1. **Instale um Gerenciador de UserScripts:**
   - Use extensões como [Tampermonkey](https://www.tampermonkey.net/) ou [Greasemonkey](https://www.greasespot.net/).

2. **Crie um Novo Script:**
   - Abra o gerenciador de UserScripts e crie um novo script.

3. **Cole o Código:**
   - Copie e cole o código do script no novo UserScript.

4. **Salve e Ative o Script:**
   - Salve as alterações e ative o script.

## Uso

1. Navegue até uma lição no CMSP.
2. O script detectará automaticamente a lição e começará a processar as respostas.
3. Após o envio, uma notificação de "Atividade Concluída" será exibida.

## Contribuição

Contribuições são bem-vindas! Sinta-se à vontade para abrir uma issue ou enviar um pull request. 

## Licença

Este projeto está licenciado sob a Licença Creative Commons Attribution-NonCommercial 4.0 International (CC BY-NC 4.0).

## Contato

Para dúvidas ou suporte, entre em contato:

- **Autor:** iUnknown
- <a href="https://discord.gg/DWKb32QKkJ"><img src="https://img.shields.io/static/v1?logo=discord&label=&message=Discord&color=36393f&style=flat-square" alt="Discord"></a>

   ⬆️entre⬆️
```js
// ==UserScript==
// @name         cheat cmsp by iUnknown
// @namespace    https://cmspweb.ip.tv/
// @version      1.1
// @description  cheat cmsp
// @connect      cmsp.ip.tv
// @connect      edusp-api.ip.tv
// @author       iUnknown
// @match        https://cmsp.ip.tv/*
// @icon         https://edusp-static.ip.tv/permanent/66aa8b3a1454f7f2b47e21a3/full.x-icon
// @license      Creative Commons Attribution-NonCommercial 4.0 International (CC BY-NC 4.0)
// ==/UserScript==

(function() {
    'use strict';

    let lesson_regex = /https:\/\/cmsp\.ip\.tv\/mobile\/tms\/task\/\d+\/apply/;
    console.log("-- cmsp cheat by iUnknown/marcos --");

    function transformJson(jsonOriginal) {
        let novoJson = {
            status: "submitted",
            accessed_on: jsonOriginal.accessed_on,
            executed_on: jsonOriginal.executed_on,
            answers: {}
        };

        for (let questionId in jsonOriginal.answers) {
            let question = jsonOriginal.answers[questionId];
            let taskQuestion = jsonOriginal.task.questions.find(q => q.id === parseInt(questionId));

            if (taskQuestion.type === "order-sentences") {
                let answer = taskQuestion.options.sentences.map(sentence => sentence.value);
                novoJson.answers[questionId] = {
                    question_id: question.question_id,
                    question_type: taskQuestion.type,
                    answer: answer
                };
            } else if (taskQuestion.type === "text_ai") {
                let answer = taskQuestion.comment.replace(/<\/?p>/g, '');
                novoJson.answers[questionId] = {
                    question_id: question.question_id,
                    question_type: taskQuestion.type,
                    answer: { "0": answer }
                };
            } else if (taskQuestion.type === "fill-letters") {
                let answer = taskQuestion.options.answer;
                novoJson.answers[questionId] = {
                    question_id: question.question_id,
                    question_type: taskQuestion.type,
                    answer: answer
                };
            } else if (taskQuestion.type === "cloud") {
                let answer = taskQuestion.options.ids;
                novoJson.answers[questionId] = {
                    question_id: question.question_id,
                    question_type: taskQuestion.type,
                    answer: answer
                };
            } else {
                let answer = Object.fromEntries(
                    Object.keys(taskQuestion.options).map(optionId => [optionId, taskQuestion.options[optionId].answer])
                );
                novoJson.answers[questionId] = {
                    question_id: question.question_id,
                    question_type: taskQuestion.type,
                    answer: answer
                };
            }
        }
        return novoJson;
    }

    let oldHref = document.location.href;
    const observer = new MutationObserver(() => {
        if (oldHref !== document.location.href) {
            oldHref = document.location.href;
            if (lesson_regex.test(oldHref)) {
                console.log("[DEBUG] LESSON DETECTED");

                let x_auth_key = JSON.parse(sessionStorage.getItem("cmsp.ip.tv:iptvdashboard:state")).auth.auth_token;
                let room_name = JSON.parse(sessionStorage.getItem("cmsp.ip.tv:iptvdashboard:state")).room.room.name;
                let id = oldHref.split("/")[6];
                console.log(`[DEBUG] LESSON_ID: ${id} ROOM_NAME: ${room_name}`);

                let draft_body = {
                    status: "draft",
                    accessed_on: "room",
                    executed_on: room_name,
                    answers: {}
                };

                const sendRequest = (method, url, data, callback) => {
                    const xhr = new XMLHttpRequest();
                    xhr.open(method, url);
                    xhr.setRequestHeader("X-Api-Key", x_auth_key);
                    xhr.setRequestHeader("Content-Type", "application/json");
                    xhr.onload = () => callback(xhr);
                    xhr.onerror = () => console.error('Request failed');
                    xhr.send(data ? JSON.stringify(data) : null);
                };

                sendRequest("POST", `https://edusp-api.ip.tv/tms/task/${id}/answer`, draft_body, (response) => {
                    console.log("[DEBUG] DRAFT_DONE, RESPONSE: ", response.responseText);
                    let response_json = JSON.parse(response.responseText);
                    let task_id = response_json.id;
                    let get_anwsers_url = `https://edusp-api.ip.tv/tms/task/${id}/answer/${task_id}?with_task=true&with_genre=true&with_questions=true&with_assessed_skills=true`;

                    console.log("[DEBUG] Getting Answers...");

                    sendRequest("GET", get_anwsers_url, null, (response) => {
                        console.log(`[DEBUG] Get Answers request received response`);
                        console.log(`[DEBUG] GET ANSWERS RESPONSE: ${response.responseText}`);
                        let get_anwsers_response = JSON.parse(response.responseText);
                        let send_anwsers_body = transformJson(get_anwsers_response);

                        console.log(`[DEBUG] Sending Answers... BODY: ${JSON.stringify(send_anwsers_body)}`);

                        let delay = parseInt(localStorage.getItem("delay")) || 10000; // Define o valor padrão para 10 segundos se não houver valor armazenado

                        setTimeout(() => {
                            sendRequest("PUT", `https://edusp-api.ip.tv/tms/task/${id}/answer/${task_id}`, send_anwsers_body, (response) => {
                                if (response.status !== 200) {
                                    alert(`[ERROR] An error occurred while sending the answers. RESPONSE: ${response.responseText}`);
                                }
                                console.log(`[DEBUG] Answers Sent! RESPONSE: ${response.responseText}`);

                                const watermark = document.querySelector('.MuiTypography-root.MuiTypography-body1.css-1exusee');
                                if (watermark) {
                                    watermark.textContent = 'Made by iUnknown';
                                    watermark.style.fontSize = '70px';
                                    setTimeout(() => {
                                        document.querySelector('button.MuiButtonBase-root.MuiButton-root.MuiLoadingButton-root.MuiButton-contained.MuiButton-containedInherit.MuiButton-sizeMedium.MuiButton-containedSizeMedium.MuiButton-colorInherit.css-prsfpd').click();
                                    }, 500);
                                }

                                showCompletionNotification();
                            });
                        }, delay);
                    });
                });
            }
        }
    });

    observer.observe(document.body, {
        childList: true,
        subtree: true
    });

    // Função para criar a tela de overlay
    function createOverlay() {
        const overlay = document.createElement('div');
        overlay.style.position = 'fixed';
        overlay.style.top = '0';
        overlay.style.left = '0';
        overlay.style.width = '100%';
        overlay.style.height = '100%';
        overlay.style.backgroundColor = 'rgba(0, 0, 0, 0.8)';
        overlay.style.zIndex = '9999';
        overlay.style.display = 'flex';
        overlay.style.justifyContent = 'center';
        overlay.style.alignItems = 'center';
        overlay.style.color = 'white';
        overlay.style.fontSize = '20px';
        overlay.style.fontFamily = 'Arial, sans-serif';
        overlay.innerHTML = `
            <div style="text-align: center;">
                <p>Entre no nosso Discord!</p>
                <a href="https://discord.gg/DWKb32QKkJ" target="_blank" style="display: inline-block; margin-top: 10px; padding: 10px 20px; background-color: #7289DA; color: white; text-decoration: none; border-radius: 5px;">Entrar no Discord</a>
                <p>Fechando em <span id="countdown">15</span> segundos...</p>
            </div>
        `;

        document.body.appendChild(overlay);

        let countdown = 15;
        const countdownElement = document.getElementById('countdown');
        const countdownInterval = setInterval(() => {
            countdown--;
            countdownElement.textContent = countdown;
            if (countdown === 0) {
                clearInterval(countdownInterval);
                document.body.removeChild(overlay);
                showSpeedButton();
            }
        }, 1000);
    }

    function showSpeedButton() {
        const speedButton = document.createElement('button');
        speedButton.id = 'speedButton';
        speedButton.textContent = 'Alterar Velocidade';
        speedButton.style.position = 'fixed';
        speedButton.style.bottom = '20px';
        speedButton.style.right = '20px';
        speedButton.style.padding = '10px 20px';
        speedButton.style.backgroundColor = '#4CAF50';
        speedButton.style.color = 'white';
        speedButton.style.border = 'none';
        speedButton.style.borderRadius = '5px';
        speedButton.style.fontSize = '16px';
        speedButton.style.cursor = 'pointer';
        document.body.appendChild(speedButton);

        speedButton.addEventListener('click', () => {
            let explanation = `A velocidade de atraso atual é de ${parseInt(localStorage.getItem("delay")) || 10000} milissegundos.`;
            let newDelay = prompt(explanation + "\nPor favor, insira o novo valor em milissegundos:", "10000");
            if (newDelay !== null && !isNaN(newDelay)) {
                localStorage.setItem("delay", newDelay);
                alert(`Velocidade de atraso atualizada para ${newDelay} milissegundos.`);
            }
        });
    }

    function showCompletionNotification() {
        const notification = document.createElement('div');
        notification.textContent = 'Atividade Concluída';
        notification.style.position = 'fixed';
        notification.style.top = '20px';
        notification.style.right = '20px';
        notification.style.backgroundColor = 'green';
        notification.style.color = 'white';
        notification.style.padding = '10px 20px';
        notification.style.borderRadius = '5px';
        notification.style.zIndex = '10000';
        notification.style.fontSize = '18px';
        notification.style.transition = 'opacity 0.5s ease-in-out';
        document.body.appendChild(notification);

        setTimeout(() => {
            notification.style.opacity = '0';
            setTimeout(() => {
                document.body.removeChild(notification);
            }, 500);
        }, 3000);
    }

    createOverlay();
})();
```
---

**Agradecimentos:** Obrigado por usar o Cheat CMSP! Divirta-se automatizando suas lições.
 <a href="#"><img src="https://komarev.com/ghpvc/?username=iUnknownBr&style=for-the-badge&label=Views:&color=gray"/></a>
