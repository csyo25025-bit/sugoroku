#define _CRT_SECURE_NO_WARNINGS
#include"DxLib.h"
#include<stdlib.h>

const int WIDTH = 1200, HEIGHT = 720;
int cw = 0;
int pw = 0;
enum { TITLE, PLAY, OVER, CLEAR };
enum {NORMAL,GO,BACK,ONEGO,ONEBACK,START,GOAL};

const int ImgMassMax = 7;
const int KomaMax = 2;

int scene = TITLE;

//文字表示
void drawTextC(int x, int y, const char* txt, int col, int siz) {
	SetFontSize(siz);
	int width = GetDrawStringWidth(txt, -1);
	DrawString(x - width / 2, y, txt, col);
}

//サイコロ
void dice(int d) {
	int m = rand() % 6 + 1;
}

//じゃんけん
void janken() {
	int p=0;

	printf("じゃんけん\n");
	printf("最初はグーじゃんけん(グー：０、チョキ＝１，パー＝２)>>");
	//scanf("%d", &p);
	int c = rand() % 3;
	if (p == c) {
		printf("あいこ");
		printf("もう一度じゃんけんをしてください");
	}
	else if ((p == 0 && c == 1) || (p == 1 && c == 2) || (p == 2 && c == 0)) {
		printf("プレイヤーの勝ち\n");
		printf("プレイヤーが先攻です\n");
		pw = 1;
	}
	else if ((p == 0 && c == 2) || (p == 1 && c == 0) || (p == 2 && c == 1)) {
		printf("コンピュータの勝ち\n");
		printf("あなたは後攻です\n");
		cw = 1;
	}
	else {
		printf("エラーもう一度じゃんけんをしてください\n");
	}
	
}
int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nCmdShow) {
	SetWindowText("すごろく");
	SetGraphMode(WIDTH,HEIGHT , 32);
	ChangeWindowMode(TRUE);
	if (DxLib_Init() == -1)return -1;
	SetBackgroundColor(0, 0, 0);
	SetDrawScreen(DX_SCREEN_BACK);


	while (ProcessMessage()==0&&ClearDrawScreen()==0) {
		ClearDrawScreen();
		ProcessMessage();
		ScreenFlip();

		switch (scene) {
		case TITLE:
			drawTextC(WIDTH  /2, HEIGHT * 3/10, "sugoroku", 0xffffff, 80);
			drawTextC(WIDTH  /2, HEIGHT *7/10, "Press SPACE to start.", 0xffffff, 30);
			if (CheckHitKey(KEY_INPUT_SPACE)) {
				scene = PLAY;
			}
			break;

		case PLAY:
			printf("先攻後攻じゃんけん\n");
			printf("勝った方が先攻\n");
			//janken();
			break;
		}
		ScreenFlip();
	}
	DxLib_End;
	return 0;
}
