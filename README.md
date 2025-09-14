# othello_game
creating two-player othello game with c

#include <stdio.h>
#include <stdbool.h>
#define SIZE 8

void InitializeBoard(char board[SIZE][SIZE]){
    for(int i=0;i<SIZE;i++){
        for(int j=0;j<SIZE;j++){
            board[i][j]=' ';
        }
    }
    board[3][3]='X';
    board[4][4]='X';
    board[3][4]='O';
    board[4][3]='O';
}

char GetOpponent(char stone){
    if(stone=='X'){
        return 'O';
    }else if(stone=='O'){
        return 'X';
    }

}
void PrintBoard(const char board[SIZE][SIZE]){
    printf("\n ");
    for(int i=0;i<SIZE;i++){
        printf("%d ",i);
    }
    
    for(int i=0;i<SIZE;i++){
        printf("\n");
        printf("%d",i);
        for(int j=0;j<SIZE;j++){
            printf("%c ",board[i][j]);
        }
    }
}
bool IsValidPosition(int x,int y){
    return (x>=0&&x<SIZE&&y>=0&&y<SIZE);
}
bool IsValidMove(char board[SIZE][SIZE],int x,int y,char player){
    //盤に石が置かれていないか
    if(!IsValidPosition(x,y)||board[x][y]!=' '){
        return false;
    }
    char opponent=GetOpponent(player);
    //挟むべき方位８つ
    const int dirs[8][2]={
        {0,1},{0,-1},{1,0},{-1,0},{1,1},{1,-1},{-1,1},{-1,-1}
    };
    //各方位挟めるかを検証
    for (int i = 0; i <8; i++){
        int nx=x+dirs[i][0];
        int ny=y+dirs[i][1];
        //隣のマスが盤内かつ敵コマのとき
        if(!IsValidPosition(nx,ny)|| board[nx][ny]!=opponent){
            continue;
        }
        while(IsValidPosition(nx,ny)&& board[nx][ny]==opponent){
            nx+=dirs[i][0];
            ny+=dirs[i][1];
        }
        //その一つ先のコマが自分の時
        if(IsValidPosition(nx,ny)&& board[nx][ny]==player){
            return true;
        }
    }
    return false;
}

bool HasValidMove(char board[SIZE][SIZE],char player){
    for(int i=0;i<SIZE;i++){
        for(int j=0; j<SIZE;j++){
            if(IsValidMove(board,i,j,player)){
                return true; //置ける場所が一つでもあれば真
            }
            
        }
    }
    return false;
}

void FlipStones(char board[SIZE][SIZE],int x,int y,char player){
    char opponent=GetOpponent(player);
        const int dirs[8][2]={
            {0,1},{0,-1},{1,0},{-1,0},{1,1},{1,-1},{-1,1},{-1,-1}
        };
        for (int i = 0; i <8; i++){
            int nx=x+dirs[i][0];
            int ny=y+dirs[i][1];
           int stones_to_flip = 0; // この方向にひっくり返す石の数

        // 相手の石が続く限り進み、その数を数える
        while (IsValidPosition(nx, ny) && board[nx][ny] == opponent) {
            stones_to_flip++;
            nx += dirs[i][0];
            ny += dirs[i][1];
        }

        // 進んだ先が自分の石で、かつ、ひっくり返す石が1個以上ある場合
        if (IsValidPosition(nx, ny) && board[nx][ny] == player && stones_to_flip > 0) {
            
            int flip_x = x + dirs[i][0];
            int flip_y = y + dirs[i][1];

            // stones_to_flip の回数だけループを回す
            for (int j = 0; j < stones_to_flip; j++) {
                board[flip_x][flip_y] = player;
                flip_x += dirs[i][0];
                flip_y += dirs[i][1];
            }
        }
    }
}
bool PlaceStone(char board[SIZE][SIZE],int x,int y,char player){
    if(IsValidMove(board,x,y,player)){
        board[x][y]=player;
        FlipStones(board,x,y,player);
        return true;
    }else{
        printf("Invalid move! Try again.\n");
        return false;
    }
}

void CountStones(const char board[SIZE][SIZE],int *x_count,int *o_count){
    for (int i = 0; i < SIZE; i++){
        for (int j = 0; j < SIZE; j++){
            if(board[i][j]=='X'){
                *x_count+=1;
            }else if(board[i][j]=='O'){
                *o_count+=1;
            }
        }
        
    }
    

}

int main() {
    printf("＝＝オセロゲーム＝＝\n");
    printf("座標は '行 列' の順番で入力してください\n\n");

    char board[SIZE][SIZE];
    InitializeBoard(board);
    PrintBoard(board);

    char player = 'X'; // 最初は 'X' (黒) のプレイヤーから
    int pass_count = 0;
    int turn_count = 0;

    // ゲームループ (盤面が埋まるか、両者パスするまで続ける)
    while (turn_count < SIZE * SIZE - 4 && pass_count < 2) {
        
        if (!HasValidMove(board, player)) {
            printf("\n\nプレイヤー %c は置ける場所がありません。パスします。", player);
            pass_count++;
            player = GetOpponent(player); // 相手のターンへ
            continue; // ループの最初に戻る
        } else {
            pass_count = 0; // 置ける場所があったらパス状態をリセット
        }
        
        int x, y;
        while (true) {
            printf("\n\nプレイヤー %c のターンです。座標を入力してください: ", player);
            
            scanf("%d %d", &x, &y);
            // 入力された手が有効かチェック
            if (IsValidMove(board, x, y, player)) {
                break; // 有効な手なので入力ループを抜ける
            } else {
                printf("その場所には置けません。別の場所を選んでください。");
            }
        }

        PlaceStone(board, x, y, player);
        turn_count++; // ターン数をカウント
        PrintBoard(board);

        int x_count = 0;
        int o_count = 0;
        CountStones(board, &x_count, &o_count);
        printf("\n-- 石の数 -- X: %d個, O: %d個", x_count, o_count);
        if (x_count>o_count){
            printf("　・・・Xが優勢です！\n");
        }else if(x_count<o_count){
            printf("　・・・Oが優勢です！\n");
        }else{
            printf("　・・・同点です！\n");
        }

        
        player = GetOpponent(player);
    }

    printf("\n\n====== ゲーム終了 ======\n");
    int x_final_count = 0;
    int o_final_count = 0;
    CountStones(board, &x_final_count, &o_final_count);

    printf("最終結果 - X: %d個, O: %d個\n", x_final_count, o_final_count);
    if (x_final_count > o_final_count) {
        printf("プレイヤー X の勝利です！\n");
    } else if (o_final_count > x_final_count) {
        printf("プレイヤー O の勝利です！\n");
    } else {
        printf("引き分けです！\n");
    }

    return 0;
}
