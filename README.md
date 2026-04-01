# sugoroku
#include"DxLib.h"
#include<stdlib.h>


//マス設定
const int MASS_MAX = 5;
int imgMass[MASS_MAX];
const int MASS_W[MASS_MAX] = { 10,10,10,10,10 };
const int MASS_H[MASS_MAX] = { 10,10,10,10,10 };


void drawMass(int x, int y, int type) {
	DrawGraph(x - MASS_W[type] / 2, y - MASS_H[type] / 2, imgMass[type], TRUE);
}

//サイコロ
void dice(void) {
	printf("サイコロを振ります");
	int DICE = rand() % 6 + 1;
	printf("サイコロの目は・・・%d", DICE);
}


int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nCmdShow) {

	//タイトル画面
	const int WIDTH = 720, HEIGHT = 640;
	SetWindowText("すごろく");
	SetGraphMode(WIDTH, HEIGHT, 32);
	ChangeWindowMode(TRUE);
	if (DxLib_Init() == -1)return -1;
	SetBackgroundColor(0, 0, 0);
	SetDrawScreen(DX_SCREEN_BACK);

	//マスの種類
	imgMass[0] = LoadGraph("IMG_E0321.JPG");
	imgMass[1] = LoadGraph("IMG_E0322.JPG");
	imgMass[2] = LoadGraph("IMG_E0323.JPG");
	imgMass[3] = LoadGraph("IMG_E0324.JPG");
	imgMass[4] = LoadGraph("IMG_E0325.JPG");
	imgMass[5] = LoadGraph("IMG_E0326.JPG");
	imgMass[6] = LoadGraph("IMG_E0327.JPG");

	//コマ
	imgMass[6] = LoadGraph("IMG_E0328.JPG");
	imgMass[7] = LoadGraph("IMG_E0329.JPG");


	int dx = 0, dy = 0;
	//プレイヤ―
	void player() {
		dx += 20;
		dice();
		drawMass(10 + dx, 10, Mass[6]);
	}


	//ゲーム進行に関する変数

	enum { TITLE, PLAY, OVER };
	int scene = TITLE;
	int timer = 0;
	
	while (1) {
		ClearDrawScreen();
		timer++;

		switch (scene) {
		case TITLE://タイトル
			drawText(160, 160, 0xffffff, "すごろく", 0, 100);
	}
	
}
