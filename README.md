#include <iostream>
#include <fstream>
#include <string>
#include <vector>
#include <sstream>
#include <algorithm>
#include <cctype>
#include <map>

using namespace std;

// Estrutura para representar uma questão
struct Questao {
    string pergunta;
    string opcoes[4];
    char respostaCorreta;
};

// Estrutura para representar um jogador no ranking
struct Jogador {
    string nome;
    int pontuacao;
};

// Função para ler as perguntas do arquivo
vector<Questao> carregarPerguntas(const string& nomeArquivo) {
    vector<Questao> perguntas;
    ifstream arquivo(nomeArquivo);
    string linha;
    Questao questaoAtual;
    int linhaAtual = 0;

    if (!arquivo.is_open()) {
        cerr << "Erro: Arquivo '" << nomeArquivo << "' nao encontrado." << endl;
        exit(1);
    }

    while (getline(arquivo, linha)) {
        if (linha.empty()) {
            if (linhaAtual != 6) {
                cerr << "Erro: Formato invalido no arquivo '" << nomeArquivo << "'." << endl;
                exit(1);
            }
            perguntas.push_back(questaoAtual);
            linhaAtual = 0;
            continue;
        }

        switch (linhaAtual) {
            case 0:
                questaoAtual.pergunta = linha;
                break;
            case 1:
                questaoAtual.opcoes[0] = linha;
                break;
            case 2:
                questaoAtual.opcoes[1] = linha;
                break;
            case 3:
                questaoAtual.opcoes[2] = linha;
                break;
            case 4:
                questaoAtual.opcoes[3] = linha;
                break;
            case 5:
                if (linha.length() == 1 && toupper(linha[0]) >= 'A' && toupper(linha[0]) <= 'D') {
                    questaoAtual.respostaCorreta = toupper(linha[0]);
                } else {
                    cerr << "Erro: Formato invalido de resposta no arquivo '" << nomeArquivo << "'." << endl;
                    exit(1);
                }
                break;
        }
        linhaAtual++;
    }

    if (linhaAtual != 0) {
        cerr << "Erro: Formato invalido no final do arquivo '" << nomeArquivo << "'." << endl;
        exit(1);
    }

    if (perguntas.empty()) {
        cerr << "Erro: Arquivo '" << nomeArquivo << "' vazio." << endl;
        exit(1);
    }

    arquivo.close();
    return perguntas;
}

// Função para exibir uma pergunta
void exibirPergunta(const Questao& questao) {
    cout << questao.pergunta << endl;
    for (const string& opcao : questao.opcoes) {
        cout << opcao << endl;
    }
    cout << "Digite sua resposta (A-D): ";
}

// Função para obter a resposta do usuário
char obterRespostaUsuario() {
    string resposta;
    cin >> resposta;
    if (resposta.length() == 1 && toupper(resposta[0]) >= 'A' && toupper(resposta[0]) <= 'D') {
        return toupper(resposta[0]);
    } else {
        cout << "Opcao invalida. Digite novamente (A-D): ";
        return obterRespostaUsuario();
    }
}

// Função para carregar o ranking do arquivo
map<string, int> carregarRanking(const string& nomeArquivo) {
    map<string, int> ranking;
    ifstream arquivo(nomeArquivo);
    string linha;
    string nome;
    int pontuacao;

    if (arquivo.is_open()) {
        while (getline(arquivo, linha)) {
            stringstream ss(linha);
            if (ss >> nome >> pontuacao) {
                ranking[nome] = pontuacao;
            }
        }
        arquivo.close();
    }
    return ranking;
}

// Função para salvar o ranking no arquivo
void salvarRanking(const string& nomeArquivo, const map<string, int>& ranking) {
    ofstream arquivo(nomeArquivo);
    if (arquivo.is_open()) {
        vector<pair<string, int>> rankingOrdenado(ranking.begin(), ranking.end());
        sort(rankingOrdenado.begin(), rankingOrdenado.end(), [](const pair<string, int>& a, const pair<string, int>& b) {
            return a.second > b.second;
        });

        for (const auto& jogador : rankingOrdenado) {
            arquivo << jogador.first << " " << jogador.second << endl;
        }
        arquivo.close();
    } else {
        cerr << "Erro ao salvar o ranking no arquivo '" << nomeArquivo << "'." << endl;
    }
}

// Função para exibir o ranking
void exibirRanking(const map<string, int>& ranking) {
    cout << "\n--- Ranking ---" << endl;
    vector<pair<string, int>> rankingOrdenado(ranking.begin(), ranking.end());
    sort(rankingOrdenado.begin(), rankingOrdenado.end(), [](const pair<string, int>& a, const pair<string, int>& b) {
        return a.second > b.second;
    });

    for (const auto& jogador : rankingOrdenado) {
        cout << jogador.first << " " << jogador.second << endl;
    }
    cout << "---------------" << endl;
}

int main() {
    vector<Questao> perguntas = carregarPerguntas("perguntas.txt");
    int pontuacao = 0;

    for (const auto& questao : perguntas) {
        exibirPergunta(questao);
        char respostaUsuario = obterRespostaUsuario();
        if (respostaUsuario == questao.respostaCorreta) {
            pontuacao++;
        }
    }

    cout << "\nQuiz finalizado!" << endl;
    cout << "Voce acertou " << pontuacao << " de " << perguntas.size() << " perguntas!" << endl;

    string nomeUsuario;
    cout << "\nDigite seu primeiro nome: ";
    cin >> nomeUsuario;

    map<string, int> ranking = carregarRanking("ranking.txt");

    if (ranking.count(nomeUsuario)) {
        if (pontuacao > ranking[nomeUsuario]) {
            ranking[nomeUsuario] = pontuacao;
            cout << "Ranking atualizado para " << nomeUsuario << "!" << endl;
        } else {
            cout << "Sua pontuacao nao superou o ranking existente para " << nomeUsuario << "." << endl;
        }
    } else {
        ranking[nomeUsuario] = pontuacao;
        cout << "Jogador " << nomeUsuario << " adicionado ao ranking!" << endl;
    }

    salvarRanking("ranking.txt", ranking);
    exibirRanking(ranking);

    return 0;
}
