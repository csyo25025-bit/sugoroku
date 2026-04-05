#define _CRT_SECURE_NO_WARNINGS
#include "DxLib.h"
#include <stdlib.h>
#include <time.h>
#include <stdio.h> 

const int WIDTH = 1200, HEIGHT = 720;
int cw = 0;
int pw = 0;
enum { TITLE, JANKEN, PLAY, OVER, CLEAR };
enum { NORMAL, GO, BACK, ONEGO, ONEBACK, START, GOAL };

const int ImgMassMax = 7;
const int KomaMax = 2;

int scene = TITLE;


int dice_val = 0;

// 文字表示
void drawTextC(int x, int y, const char* txt, int col, int siz) {
	SetFontSize(siz);
	int width = GetDrawStringWidth(txt, -1);
	DrawString(x - width / 2, y, txt, col);
}

// サイコロ
int dice() {
	return rand() % 6 + 1; // 1〜6のランダムな数字を返す
}

// こま
int player = 0, computer = 0;

int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nCmdShow) {
	SetWindowText("すごろく");
	SetGraphMode(WIDTH, HEIGHT, 32);
	ChangeWindowMode(TRUE);
	if (DxLib_Init() == -1) return -1;
	SetBackgroundColor(0, 0, 0);
	SetDrawScreen(DX_SCREEN_BACK);

	srand((unsigned int)time(NULL));

	while (ProcessMessage() == 0 && ClearDrawScreen() == 0) {

		switch (scene) {
		case TITLE:
			drawTextC(WIDTH / 2, HEIGHT * 3 / 10, "sugoroku", 0xffffff, 80);
			drawTextC(WIDTH / 2, HEIGHT * 7 / 10, "Press SPACE to start.", 0xffffff, 30);
			if (CheckHitKey(KEY_INPUT_SPACE)) {
				scene = JANKEN;
			}
			break;

		case JANKEN:
			if (pw == 0 && cw == 0) {
				drawTextC(WIDTH / 2, HEIGHT * 3 / 10, "先攻後攻じゃんけん", 0xffffff, 40);
				drawTextC(WIDTH / 2, HEIGHT * 5 / 10, "グー：Oキー、チョキ：Yキー、パー：Wキー", 0xffffff, 30);

				int p = -1;
				if (CheckHitKey(KEY_INPUT_O)) p = 0;
				if (CheckHitKey(KEY_INPUT_Y)) p = 1;
				if (CheckHitKey(KEY_INPUT_W)) p = 2;

				if (p != -1) {
					int c = rand() % 3;
					if (p == c) {
						// あいこ
					}
					else if ((p == 0 && c == 1) || (p == 1 && c == 2) || (p == 2 && c == 0)) {
						pw = 1;
					}
					else {
						cw = 1;
					}
				}
			}
			else if (pw == 1) {
				drawTextC(WIDTH / 2, HEIGHT / 2 - 40, "あなたの勝ち！先攻です", 0xffffff, 40);
				drawTextC(WIDTH / 2, HEIGHT / 2 + 40, "SPACEですごろくスタート", 0xffffff, 40);
				if (CheckHitKey(KEY_INPUT_SPACE) == 1) scene = PLAY;
			}
			else if (cw == 1) {
				drawTextC(WIDTH / 2, HEIGHT / 2 - 40, "コンピュータの勝ち！後攻です", 0xffffff, 40);
				drawTextC(WIDTH / 2, HEIGHT / 2 + 40, "SPACEですごろくスタート", 0xffffff, 40);
				if (CheckHitKey(KEY_INPUT_SPACE) == 1) scene = PLAY;
			}
			break;

		case PLAY:
			if (pw == 1) {
				
				if (dice_val == 0) {
					drawTextC(WIDTH / 2, HEIGHT / 2, "SPACEでサイコロを振って！", 0xffffff, 40);

					
					if (CheckHitKey(KEY_INPUT_SPACE) == 1) {
						dice_val = dice();
					}
				}
				// サイコロを振った後
				else {
					
					char str[256];
					sprintf(str, "出た目は【 %d 】です！", dice_val);

			
					drawTextC(WIDTH / 2, HEIGHT / 2 - 40, str, 0xffffff, 40);

					
					drawTextC(WIDTH / 2, HEIGHT / 2 + 40, "ENTERキーでコマを進める", 0xffffff, 30);
				}
			}
			break;
		}

		ScreenFlip();
	}

	DxLib_End();
	return 0;
}
