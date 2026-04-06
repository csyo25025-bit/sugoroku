#define _CRT_SECURE_NO_WARNINGS
#include "DxLib.h"
#include <stdlib.h>
#include <time.h>
#include <stdio.h> 

const int WIDTH = 1200, HEIGHT = 720;

// シーン、マスの種類、プレイヤーの種類
enum { TITLE, JANKEN, JANKEN_RESULT, PLAY, GOAL_SCENE };
enum { NORMAL, GO, BACK, ONEGO, ONEBACK, START, GOAL };
enum { PLAYER, COMUPUTER };

// ★マスを50マスに変更！
const int BOARD_LENGTH = 50;

// 画像と位置情報
int KOMA[2];
int imgMass[7];
int boardMap[BOARD_LENGTH];

int mass_x[BOARD_LENGTH];
int mass_y[BOARD_LENGTH];

// ゲーム状態
int scene = TITLE;
int dice_val = 0;
int now_pos[2] = { 0, 0 };
int current_turn = PLAYER;
int game_status = 0;

int p_hand = -1;
int c_hand = -1;

void drawTextC(int x, int y, const char* txt, int col, int siz) {
	SetFontSize(siz);
	int width = GetDrawStringWidth(txt, -1);
	DrawString(x - width / 2, y, txt, col);
}

void drawMass(int x, int y, int type) {
	DrawExtendGraph(x - 25, y - 25, x + 25, y + 25, imgMass[type], TRUE);
}

// コマを見やすくアニメーションさせる関数
void drawKoma(int x, int y, int type) {
	int offsetX = (type == PLAYER) ? -15 : 15;
	int offsetY = -15;

	if (type == current_turn) {
		int bounce = (GetNowCount() / 100) % 5;
		offsetY -= bounce;
		DrawCircle(x + offsetX, y + offsetY, 22, 0xffff00, TRUE); // オーラ
	}
	else {
		SetDrawBlendMode(DX_BLENDMODE_ALPHA, 120); // 待機中は半透明
	}

	DrawExtendGraph(x - 20 + offsetX, y - 20 + offsetY, x + 20 + offsetX, y + 20 + offsetY, KOMA[type], TRUE);
	SetDrawBlendMode(DX_BLENDMODE_NOBLEND, 0); // 描画モードを元に戻す
}

int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nCmdShow) {
	SetWindowText("50マスのすごろく(完全版)");
	SetGraphMode(WIDTH, HEIGHT, 32);
	ChangeWindowMode(TRUE);
	if (DxLib_Init() == -1) return -1;
	SetDrawScreen(DX_SCREEN_BACK);
	srand((unsigned int)time(NULL));

	KOMA[PLAYER] = LoadGraph("IMG_E0328.JPG");
	KOMA[COMUPUTER] = LoadGraph("IMG_E0329.JPG");
	imgMass[NORMAL] = LoadGraph("IMG_E0321.JPG");
	imgMass[GO] = LoadGraph("IMG_E0324.JPG");
	imgMass[BACK] = LoadGraph("IMG_E0325.JPG");
	imgMass[ONEGO] = LoadGraph("IMG_E0322.JPG");
	imgMass[ONEBACK] = LoadGraph("IMG_E0323.JPG");
	imgMass[START] = LoadGraph("IMG_E0326.JPG");
	imgMass[GOAL] = LoadGraph("IMG_E0327.JPG");

	// 50マス分の座標を自動配置 (横10マス × 縦5段)
	for (int i = 0; i < BOARD_LENGTH; i++) {
		int row = i / 10;
		int col = i % 10;
		if (row % 2 == 0) {
			mass_x[i] = 120 + col * 105;
		}
		else {
			mass_x[i] = 120 + (9 - col) * 105;
		}
		// ★50マスなので、縦の隙間を広め(100)にして画面中央にバランスよく配置
		mass_y[i] = 150 + row * 100;
	}

	// マップ生成
	boardMap[0] = START;
	boardMap[BOARD_LENGTH - 1] = GOAL;
	for (int i = 1; i < BOARD_LENGTH - 1; i++) {
		int r = rand() % 100;
		if (r < 60) boardMap[i] = NORMAL;
		else if (r < 75) boardMap[i] = ONEGO;
		else if (r < 90) boardMap[i] = ONEBACK;
		else if (r < 95) boardMap[i] = GO;
		else boardMap[i] = BACK;
	}

	while (ProcessMessage() == 0 && ClearDrawScreen() == 0 && CheckHitKey(KEY_INPUT_ESCAPE) == 0) {
		switch (scene) {
		case TITLE:
			drawTextC(WIDTH / 2, 200, "50-MASS SUGOROKU", 0xffffff, 80);
			drawTextC(WIDTH / 2, 500, "Press SPACE to start.", 0xffffff, 30);
			if (CheckHitKey(KEY_INPUT_SPACE)) scene = JANKEN;
			break;

		case JANKEN:
		{
			drawTextC(WIDTH / 2, 100, "先攻後攻じゃんけん", 0xffffff, 40);
			drawTextC(WIDTH / 2, 200, "O:グー Y:チョキ W:パー", 0xffffff, 30);
			int p = -1;
			if (CheckHitKey(KEY_INPUT_O)) p = 0;
			if (CheckHitKey(KEY_INPUT_Y)) p = 1;
			if (CheckHitKey(KEY_INPUT_W)) p = 2;
			if (p != -1) {
				int c = rand() % 3;
				p_hand = p;
				c_hand = c;

				if (p != c) {
					current_turn = ((p == 0 && c == 1) || (p == 1 && c == 2) || (p == 2 && c == 0)) ? PLAYER : COMUPUTER;
				}
				scene = JANKEN_RESULT;
				WaitTimer(200);
			}
			break;
		}

		case JANKEN_RESULT:
		{
			drawTextC(WIDTH / 2, 100, "じゃんけん結果", 0xffffff, 40);
			const char* hands[] = { "グー", "チョキ", "パー" };
			char strP[256], strC[256];
			sprintf(strP, "あなた：【 %s 】", hands[p_hand]);
			sprintf(strC, "コンピュータ：【 %s 】", hands[c_hand]);

			drawTextC(WIDTH / 2, 250, strP, 0xffffff, 30);
			drawTextC(WIDTH / 2, 320, strC, 0xffffff, 30);

			if (p_hand == c_hand) {
				drawTextC(WIDTH / 2, 450, "あいこ！ SPACEキーでもう一度！", 0xffff00, 40);
				if (CheckHitKey(KEY_INPUT_SPACE) == 1) {
					scene = JANKEN;
					WaitTimer(200);
				}
			}
			else {
				if (current_turn == PLAYER) {
					drawTextC(WIDTH / 2, 450, "あなたの勝ち！【 先攻 】です", 0x00ff00, 40);
				}
				else {
					drawTextC(WIDTH / 2, 450, "コンピュータの勝ち…【 後攻 】です", 0xff0000, 40);
				}
				drawTextC(WIDTH / 2, 600, "ENTERキーでゲームスタート！", 0xffffff, 30);

				if (CheckHitKey(KEY_INPUT_RETURN) == 1) {
					scene = PLAY;
					WaitTimer(200);
				}
			}
			break;
		}

		case PLAY:
		{
			for (int i = 0; i < BOARD_LENGTH; i++) drawMass(mass_x[i], mass_y[i], boardMap[i]);

			// ★今アクティブなコマが一番上に描画されるように順番を制御
			if (current_turn == PLAYER) {
				drawKoma(mass_x[now_pos[COMUPUTER]], mass_y[now_pos[COMUPUTER]], COMUPUTER);
				drawKoma(mass_x[now_pos[PLAYER]], mass_y[now_pos[PLAYER]], PLAYER);
			}
			else {
				drawKoma(mass_x[now_pos[PLAYER]], mass_y[now_pos[PLAYER]], PLAYER);
				drawKoma(mass_x[now_pos[COMUPUTER]], mass_y[now_pos[COMUPUTER]], COMUPUTER);
			}

			if (current_turn == PLAYER) drawTextC(WIDTH / 2, 20, "あなたの番です", 0xffffff, 30);
			else drawTextC(WIDTH / 2, 20, "コンピュータが考えています...", 0xffaa00, 30);

			if (game_status == 0) {
				if (current_turn == PLAYER) {
					drawTextC(WIDTH / 2, 670, "SPACEキーでサイコロを振る", 0xffffff, 30);
					if (CheckHitKey(KEY_INPUT_SPACE) == 1) {
						dice_val = rand() % 6 + 1;
						game_status = 1;
						WaitTimer(200);
					}
				}
				else {
					drawTextC(WIDTH / 2, 670, "コンピュータがサイコロを振ります...", 0xffaa00, 30);
					ScreenFlip();
					WaitTimer(800);
					dice_val = rand() % 6 + 1;
					game_status = 1;
				}
			}
			else if (game_status == 1) {
				char str[256];
				sprintf(str, "出た目は【 %d 】！", dice_val);
				drawTextC(WIDTH / 2, 660, str, 0xffffff, 30);

				if (current_turn == PLAYER) {
					drawTextC(WIDTH / 2, 690, "ENTERキーで進む", 0x00ff00, 20);
					if (CheckHitKey(KEY_INPUT_RETURN) == 1) {
						now_pos[current_turn] += dice_val;
						if (now_pos[current_turn] >= BOARD_LENGTH - 1) {
							now_pos[current_turn] = BOARD_LENGTH - 1;
							scene = GOAL_SCENE;
						}
						else { game_status = 2; }
						WaitTimer(200);
					}
				}
				else {
					ScreenFlip();
					WaitTimer(800);
					now_pos[current_turn] += dice_val;
					if (now_pos[current_turn] >= BOARD_LENGTH - 1) {
						now_pos[current_turn] = BOARD_LENGTH - 1;
						scene = GOAL_SCENE;
					}
					else { game_status = 2; }
				}
			}
			else if (game_status == 2) {
				int effect = boardMap[now_pos[current_turn]];
				const char* msg = (effect == ONEGO) ? "【1マス進む！】" : (effect == ONEBACK) ? "【1マス戻る…】" : (effect == GO) ? "【さらに出た目だけ進む！】" : (effect == BACK) ? "【さらに出た目だけ戻る…】" : "なにも起きない。";
				drawTextC(WIDTH / 2, 660, msg, 0xffffff, 30);

				if (current_turn == PLAYER) {
					drawTextC(WIDTH / 2, 690, "ENTERキーで交代", 0x00ff00, 20);
					if (CheckHitKey(KEY_INPUT_RETURN) == 1) {
						if (effect == ONEGO) now_pos[current_turn] += 1;
						else if (effect == ONEBACK) now_pos[current_turn] -= 1;
						else if (effect == GO) now_pos[current_turn] += dice_val;
						else if (effect == BACK) now_pos[current_turn] -= dice_val;
						if (now_pos[current_turn] < 0) now_pos[current_turn] = 0;
						if (now_pos[current_turn] >= BOARD_LENGTH - 1) { now_pos[current_turn] = BOARD_LENGTH - 1; scene = GOAL_SCENE; }
						current_turn = COMUPUTER; game_status = 0; dice_val = 0;
						WaitTimer(200);
					}
				}
				else {
					ScreenFlip();
					WaitTimer(1200);
					if (effect == ONEGO) now_pos[current_turn] += 1;
					else if (effect == ONEBACK) now_pos[current_turn] -= 1;
					else if (effect == GO) now_pos[current_turn] += dice_val;
					else if (effect == BACK) now_pos[current_turn] -= dice_val;
					if (now_pos[current_turn] < 0) now_pos[current_turn] = 0;
					if (now_pos[current_turn] >= BOARD_LENGTH - 1) { now_pos[current_turn] = BOARD_LENGTH - 1; scene = GOAL_SCENE; }
					current_turn = PLAYER; game_status = 0; dice_val = 0;
				}
			}
			break;
		}

		case GOAL_SCENE:
			drawTextC(WIDTH / 2, 300, "GOAL!!", 0xff0000, 100);
			if (now_pos[PLAYER] == BOARD_LENGTH - 1) drawTextC(WIDTH / 2, 450, "あなたの勝ちです！", 0xffffff, 50);
			else drawTextC(WIDTH / 2, 450, "コンピュータの勝ちです…", 0xffffff, 50);
			drawTextC(WIDTH / 2, 600, "ESCキーで終了", 0xaaaaaa, 30);
			break;
		}
		ScreenFlip();
	}
	DxLib_End();
	return 0;
}
