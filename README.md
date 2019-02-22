# librarySystem
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<windows.h>
#include<time.h>
#define MAXDAY 30
#define rate 0.1
char adminID[20]="1";
char adminPassword[20]="1";
char stuID[20];//全局变量后续还需要通过这个账号获取其他信息 因此要一直保留
char admID[20];
struct readerInfo*stuNode=NULL;

struct tm* timeGet()
{
    time_t tmpcal_ptr;
	struct tm *tmp_ptr = NULL;
	time(&tmpcal_ptr);//time函数返回的是一个结构体（也相当于一个大整数）
	tmp_ptr = localtime(&tmpcal_ptr);//获取当地时间,存放于结构体中
	return tmp_ptr;
}
struct time
{
    int year;
    int month;
    int day;
};
struct bookInfo
{
    char isbn[20];
    char bookName[20];
    char writer[20];
    char kind[20];
    char press[20];
    char time[20];
    int amount;
    double price;
    struct bookInfo*next;
};
struct readerInfo
{
    int no;
    char id[20];
    char readerName[20];
    char password[20];
    int maxAmount;
    int keepAmount;
    struct readerInfo*next;
};
struct borrowInfo
{
    char readerId[20];
    char readerName[20];
    char isbn[20];
    char bookName[20];
    char writer[20];
    char press[20];
    char sign[10];
    struct time borrowTime;
    struct borrowInfo*next;
};
struct returnInfo
{
    char readerId[20];
    char readerName[20];
    char isbn[20];
    char bookName[20];
    char writer[20];
    char press[20];
    struct time returnTime;
    struct returnInfo*next;
};
//借阅时间计算
int monthCount(int year,int month,int day)//计算当天是当年的第几天
{
    int sumdays=0;
    switch(month)
    {
    case 12:
        sumdays+=30;
    case 11:
        sumdays+=31;
    case 10:
        sumdays+=30;
    case 9:
        sumdays+=31;
    case 8:
        sumdays+=31;
    case 7:
        sumdays+=30;
    case 6:
        sumdays+=31;
    case 5:
        sumdays+=30;
    case 4:
        sumdays+=31;
    case 3:
        sumdays+=28;
    case 2:
        sumdays+=31;
    case 1:
        sumdays+=day;
    }
    if(((year%4==0&&year%100!=0)||(year%400==0))&&month>2)
        sumdays+=1;

    return sumdays-1;
}
int yearCount(int yearStart,int yearEnd)//计算年份差距从而计算year*366 or year*365
{
    int yearDaySum=0;
    for(int i=yearStart+1;i<yearEnd;i++)
    {
        if((yearStart%4==0&&yearStart%100!=0)||yearStart%400==0)
            yearDaySum+=366;
        else yearDaySum+=365;
    }
    return yearDaySum;
}
int dayCount(int startYear,int startMonth,int startDay,int endYear,int endMonth,int endDay)//计算两个时间之间差的天数
{
    int sum=365;
    if((startYear%4==0&&startYear%100!=0)||startYear%400==0)
        sum=366;
    int startYearDay=monthCount(startYear,startMonth,startDay);
    int endYearDay=monthCount(endYear,endMonth,endDay);
    int day=yearCount(startYear,endYear);

    if(startYear<endYear)
        return sum-startYearDay+endYearDay+day;
    else if(startYear==endYear)
        return startYearDay-startYearDay;
}
//全局变量：用于加载图书信息 读者信息 借书信息 还书信息的头结点
struct bookInfo*head=(struct bookInfo*)malloc(sizeof(bookInfo));
struct readerInfo*rhead=(struct readerInfo*)malloc(sizeof(readerInfo));
struct borrowInfo*bohead=(struct borrowInfo*)malloc(sizeof(borrowInfo));
struct returnInfo*rehead=(struct returnInfo*)malloc(sizeof(returnInfo));

void memsetAll()
{
    memset(head,0,sizeof(bookInfo));
    memset(rhead,0,sizeof(readerInfo));
    memset(bohead,0,sizeof(borrowInfo));
    memset(rehead,0,sizeof(returnInfo));

}
void adminMenu();
void bookSearchMainMenu();
void mainMenu();
void modifyMenu();
void bookadminMenu();
void readeradminMenu();
void readeradminMenuBR();
void borrowInfoMenu();
void returnInfoMenu();
void readeruserMenu();


int istime(struct bookInfo*from,char*time);
void bookModifyView(struct bookInfo*current);
void bookInfocpy(struct bookInfo*from,struct bookInfo*to);

struct bookInfo*bookGetnode();
void initialSave();
void downloadBook();


struct bookInfo *bookSearch(char*isbn,char*bookName);
void bookSave();
void bookView(struct bookInfo*head);
void bookInsert();
void bookDelete();
void bookModify();


void bookSearchName();
void bookSearchWriter();
void bookSearchPress();
void bookSearchTime();
/************************************************************/
void readerModifyView(struct readerInfo*current);
struct readerInfo*readerGetnode();
void readerInitialSave();
void readerSave(struct readerInfo*rhead);
void downloadReader();
void readerInsert();
void readerDelete();
struct readerInfo*readerSearch(char *readerID);
void readerModifyMenu();
void readerModify();
void readerView();
/***********************************************************/
void borrowView(struct borrowInfo*head);
void downloadBorrowInfo();
void borrowSearch(int type,char*info);
void borrowSearchBook();
void borrowSearchReader();
void borrowSearchWriter();
struct borrowInfo*borrowSearchUB(char*stuid,char*bookisbn);

void returnView(struct returnInfo*head);
void returnSearch(int type,char*info);
void returnSearchReader();
void returnSearchBook();
void returnSearchWriter();
void downloadReturnInfo();
/*********************************************************/
void bookReturn();
void bookBorrow();
void bookReturnAdmin();
/********************************************************/
int adminLogin();
int readerLogin();
void userModify();
void signUp();
void userLoginMenu();
/********************************************************/
void readerFree();
void bookFree();
void borrowFree();

void adminMenu()
{
    int choice;
    system("cls");
    printf("\t\t\t\t\t\t1.图书信息管理\n");
    printf("\t\t\t\t\t\t2.读者信息管理\n");
    printf("\t\t\t\t\t\t3.借阅信息管理\n\n");
    printf("\t\t\t\t\t\t9.返回上一级\n");
    printf("请选择：\n");
    scanf("%d",&choice);
    switch(choice)
    {
        case 1:
            bookadminMenu();
            break;
        case 2:
            readeradminMenu();
            break;
        case 3:
            readeradminMenuBR();
            break;
        case 9:
            mainMenu();
            break;
        default:
            printf("没有相应操作的选项！\n");
    }
}
void readeradminMenu()
{
    int choice;
    char id[20];
    struct readerInfo*current=NULL;
    do
    {
    system("cls");
    printf("\t\t\t\t\t# # # # #1.初始读者信息 # # # # #\n");
    printf("\t\t\t\t\t# # # # #2.查看读者信息 # # # # #\n");
    printf("\t\t\t\t\t# # # # #3.添加读者信息 # # # # #\n");
    printf("\t\t\t\t\t# # # # #4.删除读者信息 # # # # #\n");
    printf("\t\t\t\t\t# # # # #5.查找读者信息 # # # # #\n");
    printf("\t\t\t\t\t# # # # #6.修改读者信息 # # # # #\n");

    printf("\n\n");
    printf("\t\t\t\t\t*********9.返回上一级      *********\n");
    printf("\t\t\t\t\t*********0.返回主菜单  *********\n");

    printf("请选择：\n");
    scanf("%d",&choice);
    fflush(stdin);
    switch(choice)
    {
    case 1:
        readerInitialSave();
        break;
    case 2:
        readerView();
        break;
    case 3:
        readerInsert();
        break;
    case 4:
        readerDelete();
        break;
    case 5:
        system("cls");
        printf("请输入读者借阅号:\n");
        gets(id);
        fflush(stdin);
        current=readerSearch(id);
        if(current==NULL)
            printf("系统没有您要查找的读者信息！\n");
        else
        {
            printf("您要查询的读者信息为：\n");
            readerModifyView(current);
        }
        break;
    case 6:
        readerModify();
        break;
    case 9:
        adminMenu();
        return;
    case 0:
        mainMenu();
        return;
    }
    getchar();
    }while(choice!=0||choice!=9);
}
void readeradminMenuBR()
{
    int choice;
    do{
        system("cls");
        printf("\t\t\t\t\t# # # # #1.借书信息查询 # # # # #\n");
        printf("\t\t\t\t\t# # # # #2.还书信息查询 # # # # #\n");
        printf("\t\t\t\t\t# # # # #3.归还逾期图书 # # # # #\n");
        printf("\n\n");
        printf("\t\t\t\t\t*********9.返回上一级      *********\n");
        printf("\t\t\t\t\t*********0.返回主菜单  *********\n");
        printf("请选择：\n");
        scanf("%d",&choice);
        fflush(stdin);
        switch(choice)
        {
            case 1:
                borrowInfoMenu();
                getchar();
                break;
            case 2:
                returnInfoMenu();
                getchar();
                break;
            case 3:
                bookReturnAdmin();
                getchar();
                break;
            case 9:
                adminMenu();
                return;
            case 0:
                mainMenu();
                return;
            default:
                printf("没有相应的选项！");
        }
    }while(choice!=0||choice!=9);
}
void bookadminMenu()
{
    int choice;
    do
    {
        system("cls");
        printf("\t\t\t\t\t# # # # #1.图书初始化  # # # # #\n");
        printf("\t\t\t\t\t# # # # #2.查看全部图书# # # # #\n");
        printf("\t\t\t\t\t# # # # #3.添加新的图书# # # # #\n");
        printf("\t\t\t\t\t# # # # #4.删除图书    # # # # #\n");
        printf("\t\t\t\t\t# # # # #5.查找图书    # # # # #\n");
        printf("\t\t\t\t\t# # # # #6.修改图书    # # # # #\n");

        printf("\n\n");
        printf("\t\t\t\t\t*********9.上一级      *********\n");
        printf("\t\t\t\t\t*********0.返回主菜单  *********\n");

        printf("请选择：\n");
        scanf("%d",&choice);
        fflush(stdin);
        switch(choice)
        {
        case 1:
            initialSave();
            break;
        case 2:
            bookView(head);
            break;
        case 3:
            bookInsert();
            break;
        case 4:
            bookDelete();
            break;
        case 5:
            bookSearchMainMenu();
            break;
        case 6:
            bookModify();
            break;
        case 9:
            adminMenu();
            return ;
        case 0:
            mainMenu();
            return ;
        default:
            printf("没有相应选项！");
        }
        getchar();
    }while(choice!=0||choice!=9);
}
void bookSearchMainMenu()
{
    int choice;
    system("cls");
    printf("\t\t\t\t\t       1.按书籍名查询          \n");
    printf("\t\t\t\t\t       2.按作者名查询          \n");
    printf("\t\t\t\t\t       3.按出版社名查询        \n");
    printf("\t\t\t\t\t       4.按出版时间查询        \n");
    printf("\n");

    printf("请选择！\n");
    scanf("%d",&choice);
    fflush(stdin);
    switch(choice)
    {
    case 1:
        bookSearchName();
        break;
    case 2:
        bookSearchWriter();
        break;
    case 3:
        bookSearchPress();
        break;
    case 4:
        bookSearchTime();
        break;
    default:
        printf("没有相应查找方式！\n");
    }
    getchar();
}
void mainMenu()
{
    int choice;
    system("cls");
    printf("\t\t\t\t\t*************1.管理员**************\n");
    printf("\t\t\t\t\t*************2.读者****************\n\n");
    printf("\t\t\t\t\t*************0.退出****************\n");
    scanf("%d",&choice);
    fflush(stdin);
    switch(choice)
    {
    case 1:
        if(adminLogin())
            adminMenu();
        else {
            printf("密码或账号错误！按任意键继续！\n");
            getchar();
            system("cls");
            mainMenu();
        }
        return;
    case 2:
        userLoginMenu();
        break;
    case 0:
        return;
    }

}
void borrowInfoMenu()
{
    int choice;
    system("cls");
    printf("\t\t\t\t\t1.按书名查借书信息\n");
    printf("\t\t\t\t\t2.按读者查询借书信息\n");
    printf("\t\t\t\t\t3.按作者查询借书信息\n\n");

    printf("请选择:\n");
    scanf("%d",&choice);
    fflush(stdin);
    switch(choice)
    {
        case 1:
            borrowSearchBook();
            break;
        case 2:
            borrowSearchReader();
            break;
        case 3:
            borrowSearchWriter();
            break;
        default:
            printf("没有相应的选项\n");
    }
}
void returnInfoMenu()
{
    int choice;
    system("cls");
    printf("\t\t\t\t\t1.按书名查询还书信息\n");
    printf("\t\t\t\t\t2.按读者查询还书信息\n");
    printf("\t\t\t\t\t3.按作者查询还书信息\n\n");

    printf("\t\t\t\t\t9.返回上一级\n");
    printf("\t\t\t\t\t0.返回主菜单\n");
    printf("请选择:\n");
    scanf("%d",&choice);
    fflush(stdin);
    switch(choice)
    {
        case 1:
            returnSearchBook();
            getchar();
            break;
        case 2:
            returnSearchReader();
            getchar();
            break;
        case 3:
            returnSearchWriter();
            getchar();
            break;
        case 9:
            readeradminMenuBR();
            return;
        case 0:
            mainMenu();
            return;
        default:
            printf("没有相应的选项\n");
    }
}

void readeruserMenu()
{
    int choice;
    do
    {
        system("cls");
        printf("\t\t\t\t\t# # # # #1.图书查找# # # # #\n");
        printf("\t\t\t\t\t# # # # #2.借书功能# # # # #\n");
        printf("\t\t\t\t\t# # # # #3.还书功能# # # # #\n");
        printf("\t\t\t\t\t# # # # #4.修改信息# # # # #\n\n");
        printf("\t\t\t\t\t*********9.返回上一级*******\n");
        printf("请选择：\n");
        scanf("%d",&choice);
        fflush(stdin);
        switch(choice)
        {
        case 1:
            bookSearchMainMenu();
            getchar();
            break;
        case 2:
            bookBorrow();
            getchar();
            break;
        case 3:
            bookReturn();
            getchar();
            break;
        case 4:
            userModify();
            getchar();
            break;
        case 9:
            mainMenu();
            return;
        }
    }while(choice!=9);
}

void userLoginMenu()
{
    int choice;
    system("cls");
    printf("\t\t\t\t\t欢迎使用图书图书借阅系统\n\n");
    printf("\t\t\t\t\t     1.读者登陆\n");
    printf("\t\t\t\t\t     2.读者注册\n\n");
    printf("\t\t\t\t\t     0.返回主菜单\n");
    printf("请选择：\n");
    scanf("%d",&choice);
    fflush(stdin);
    switch(choice)
    {
    case 1:
        if(readerLogin())
           readeruserMenu();
        else
          getchar();
          userLoginMenu();
        break;
    case 2:
        signUp();
        getchar();
        userLoginMenu();
        break;
    case 0:
        mainMenu();
        return ;
    }
    return ;
}

struct bookInfo *bookSearch(char*isbn)
{
    struct bookInfo*current=head->next;
    while(current!=NULL)
    {
        if(strcmp(current->isbn,isbn)==0)
            return current;
        current=current->next;
    }
    return NULL;
}
int istime(struct bookInfo*from,char*time)//判断时间是否准确，时间支持三种格式
{
    int i=0;
    while(i<strlen(time))
    {
        if(from->time[i]!=time[i])
        return 0;
        if(from->time[i]==time[i])
            i++;
    }
    if(i==strlen(time))
        return 1;
    return 0;
}
void bookModifyView(struct bookInfo*current)//检查修改后的信息是否准确
{
    if(current!=NULL)
    {
       printf("编号            书名                 作者名        分类号     出版单位        出版时间      库存数量     价格\n\n");
       printf("%-15s %-20s %-15s %-8s %-15s %-15s %-10d %-10.3lf\n",current->isbn,current->bookName,current->writer
               ,current->kind,current->press,current->time,current->amount,current->price);
    }
}
void bookInfocpy(struct bookInfo*from,struct bookInfo*to)//信息查询时创建新的链表将信息复制
{
    strcpy(to->isbn,from->isbn);
    strcpy(to->bookName,from->bookName);
    strcpy(to->writer,from->writer);
    strcpy(to->kind,from->kind);
    strcpy(to->press,from->press);
    strcpy(to->time,from->time);
    to->amount=from->amount;
    to->price=from->price;
}
struct bookInfo*getnode()
{
    struct bookInfo*current=(struct bookInfo*)malloc(sizeof(struct bookInfo));
    memset(current,0,sizeof(bookInfo));
    printf("请输入图书编号：\n");
    gets(current->isbn);
    fflush(stdin);
    printf("请输入图书名称：\n");
    gets(current->bookName);
    fflush(stdin);
    printf("请输入作者姓名：\n");
    gets(current->writer);
    fflush(stdin);
    printf("请输入分类号：\n");
    gets(current->kind);
    fflush(stdin);
    printf("请输入出版单位：\n");
    gets(current->press);
    fflush(stdin);
    printf("请输入出版时间：\n");
    gets(current->time);
    fflush(stdin);
    printf("请输入库存数量：\n");
    scanf("%d",&current->amount);
    fflush(stdin);
    printf("请输入价格：\n");
    scanf("%lf",&current->price);
    fflush(stdin);

    return current;
}
void initialSave()//格式化（首次使用）输入信息从而初始化
{
    FILE *fp;
    bookFree();
    head->next=NULL;
    struct bookInfo*tail=head;
    struct bookInfo*newNode=NULL;
    char judege[20];
    system("cls");
    if ((fp = fopen("bookinformation.txt", "w+"))==NULL)
	{
		printf("不能打开书目信息文件夹！");
		return ;
	}
	do
    {
        system("cls");
        newNode=getnode();
        while(tail->next!=NULL&&strcmp(tail->next->isbn,newNode->isbn)<0)
            tail=tail->next;
        newNode->next=tail->next;
        tail->next=newNode;
        printf("按任意键继续,quit退出！");
        gets(judege);
        fflush(stdin);
    }while(strcmp(judege,"quit")!=0);
    printf("图书初始化完成！\n");
    return ;
}
void downloadBook()//从书籍目录读取书籍信息
{
    struct bookInfo*tail=head;
    struct bookInfo*current=NULL;
    FILE*fp;
    if ((fp = fopen("bookinformation.txt", "r+"))==NULL)
		printf("不能打开书目信息文件夹！");
    else{
        while(!feof(fp))
        {
            current=(struct bookInfo*)malloc(sizeof(bookInfo));
            memset(current,0,sizeof(bookInfo));
            if(fread(current,sizeof(bookInfo),1,fp)!=0)
            {
                tail->next=current;
                tail=current;
            }
            else free(current);
        }
    }
    return ;
    fclose(fp);
}
void bookSave(struct bookInfo*head)
{
    FILE*fp;
    struct bookInfo*current=head->next;
    struct bookInfo*temp=NULL;
    if ((fp = fopen("bookinformation.txt", "w"))==NULL)
	{
		printf("不能打开书目信息文件夹，书籍信息保存失败！");
		return ;
	}
    while(current!=NULL)
    {
        fwrite(current,sizeof(bookInfo),1,fp);
        current=current->next;
    }
    fclose(fp);
    printf("图书信息保存成功！\n");
}
void bookView(struct bookInfo*head)//浏览全部的图书信息
{
    struct bookInfo*current=head->next;
    printf("编号            书名                 作者名        分类号     出版单位        出版时间      库存数量     价格\n\n");
    while(current!=NULL)
    {
        printf("%-15s %-20s %-15s %-8s %-15s %-15s %-10d %-10.3lf\n",current->isbn,current->bookName,current->writer
               ,current->kind,current->press,current->time,current->amount,current->price);
        current=current->next;
    }
    return ;
}
void bookInsert()//插入新的书籍
{
    struct bookInfo*Newnode=NULL;
    struct bookInfo*current=head;
    char judege[20];
    do{
        system("cls");
        printf("请输入插入书籍的相关信息：\n");
        fflush(stdin);
        Newnode=getnode();
        while(current->next!=NULL&&strcmp(current->next->isbn,Newnode->isbn)<0)
            current=current->next;
        Newnode->next=current->next;
        current->next=Newnode;
        printf("按任意键继续添加书籍，输入quit退出！");
        gets(judege);
        fflush(stdin);
    }while(strcmp(judege,"quit")!=0);
    return ;
}
void bookDelete()//当有两个限制条件时，书籍唯一
{
    struct bookInfo*current=head;
    struct bookInfo*temp=NULL;
    char bookName[20],isbn[20],judege[20];
    printf("请输入待删除的图书编号：\n");
    gets(isbn);
    fflush(stdin);
    if(current->next==NULL)
        printf("书籍目录为空，无法进行删除操作！");
    else if((temp=bookSearch(isbn))!=NULL)
    {
        printf("待删除的信息为：\n");
        bookModifyView(temp);
        printf("按任意键删除，输入quit退出！\n");
        gets(judege);
        if(strcmp(judege,"quit")==0)
        {
            bookadminMenu();
            return ;
        }
        while(current->next!=NULL)
        {
            if(strcmp(current->next->isbn,isbn)==0)
            {
                temp=current->next;
                current->next=temp->next;
                free(temp);
                return ;
            }
            current=current->next;
        }
    }
}
void modifyMenu()
{

    printf("\t\t\t\t\t             1.修改编号\n");
    printf("\t\t\t\t\t             2.修改书名\n");
    printf("\t\t\t\t\t             3.修改作者名\n");
    printf("\t\t\t\t\t             4.修改分类号\n");
    printf("\t\t\t\t\t             5.修改出版单位\n");
    printf("\t\t\t\t\t             6.修改出版时间\n");
    printf("\t\t\t\t\t             7.修改库存数量\n");
    printf("\t\t\t\t\t             8.修改价格\n");
}
void bookModify()
{
    struct bookInfo*current=NULL;
    int choice;
    char isbn[20],bookName[20],judege[20],judege2[20];
    system("cls");
    printf("请输入图书编号：\n");
    gets(isbn);
    fflush(stdin);
    current=bookSearch(isbn);
    if(current==NULL)
        printf("没有您要修改的的图书！\n");
    else
    {
        printf("待修改的图书信息为：\n");
        bookModifyView(current);
        printf("按任意键继续，输入quit退出修改！\n");
        gets(judege);
        fflush(stdin);
        if(strcmp(judege,"quit")==0)
            return ;
        else
        {
            modifyMenu();
            do{
                printf("请选择修改项目：\n");
                scanf("%d",&choice);
                fflush(stdin);
                switch(choice)
                {
                case 1:
                    printf("请输入修改后的编号：\n");
                    scanf("%s",&current->isbn);
                    fflush(stdin);
                    break;
                case 2:
                    printf("请输入修改后的书名：\n");
                    scanf("%s",&current->bookName);
                    fflush(stdin);
                    break;
                case 3:
                    printf("请输入修改后的作者名：\n");
                    scanf("%s",&current->writer);
                    fflush(stdin);
                    break;
                case 4:
                    printf("请输入修改后的分类号：\n");
                    scanf("%s",&current->kind);
                    fflush(stdin);
                    break;
                case 5:
                    printf("请输入修改后的出版单位：\n");
                    scanf("%s",&current->press);
                    fflush(stdin);
                    break;
                case 6:
                    printf("请输入修改后的出版时间：\n");
                    scanf("%s",&current->time);
                    fflush(stdin);
                    break;
                case 7:
                    printf("请输入修改后的库存数量：\n");
                    scanf("%d",&current->amount);
                    fflush(stdin);
                    break;
                case 8:
                    printf("请输入修改后的价格：\n");
                    scanf("%lf",&current->price);
                    fflush(stdin);
                    break;
                default:
                    printf("没有相应的选项！\n");
                }
                printf("修改后的信息为；\n");
                bookModifyView(current);
                printf("按任意键继续修改当前书籍，输入quit退出修改！");
                gets(judege2);
                fflush(stdin);
            }while(strcmp(judege2,"quit")!=0);

    }
    printf("修改完成！\n");
    return ;
    }

}
void bookSearchName()
{
    struct bookInfo*current=head->next;
    struct bookInfo*newNode=NULL;
    struct bookInfo*bname_head=(struct bookInfo*)malloc(sizeof(bookInfo));
    memset(bname_head,0,sizeof(bookInfo));
    struct bookInfo*tail=bname_head;
    char bname[20];
    system("cls");
    printf("请输入待查找书籍名称：\n");
    gets(bname);
    fflush(stdin);
    while(current!=NULL)
    {
        if(strcmp(current->bookName,bname)==0)
        {
            newNode=(struct bookInfo*)malloc(sizeof(bookInfo));
            memset(newNode,0,sizeof(bookInfo));
            bookInfocpy(current,newNode);
            while(tail->next!=NULL)
                tail=tail->next;
            tail->next=newNode;
            newNode->next=NULL;
        }
        current=current->next;
    }
    if(bname_head->next==NULL)
        printf("图书馆没有相应图书！\n");
    else bookView(bname_head);
    return ;
}
void bookSearchWriter()
{
    struct bookInfo*current=head->next;
    struct bookInfo*newNode=NULL;
    struct bookInfo*wname_head=(struct bookInfo*)malloc(sizeof(bookInfo));
    memset(wname_head,0,sizeof(bookInfo));
    struct bookInfo*tail=wname_head;
    char wname[20];
    system("cls");
    printf("请输入待查找作者名：\n");
    gets(wname);
    fflush(stdin);
    while(current!=NULL)
    {
        if(strcmp(current->writer,wname)==0)
        {
            newNode=(struct bookInfo*)malloc(sizeof(bookInfo));
            memset(newNode,0,sizeof(bookInfo));
            bookInfocpy(current,newNode);
            while(tail->next!=NULL)
                tail=tail->next;
            tail->next=newNode;
            newNode->next=NULL;
        }
        current=current->next;
    }
    //这部分可以在search函数外面，直接调用这个函数从而查看相关信息
    if(wname_head->next==NULL)
        printf("图书馆没有相应作者的图书！\n");
    else bookView(wname_head);
    return ;
}
void bookSearchPress()
{
    struct bookInfo*current=head->next;
    struct bookInfo*newNode=NULL;
    struct bookInfo*pname_head=(struct bookInfo*)malloc(sizeof(bookInfo));
    memset(pname_head,0,sizeof(bookInfo));
    struct bookInfo*tail=pname_head;
    char pname[20];
    system("cls");
    printf("请输入待查找出版社名：\n");
    gets(pname);
    fflush(stdin);
    while(current!=NULL)
    {
        if(strcmp(current->press,pname)==0)
        {
            newNode=(struct bookInfo*)malloc(sizeof(bookInfo));
            memset(newNode,0,sizeof(bookInfo));
            bookInfocpy(current,newNode);
            while(tail->next!=NULL)
                tail=tail->next;
            tail->next=newNode;
            newNode->next=NULL;
        }
        current=current->next;
    }
    if(pname_head->next==NULL)
        printf("图书馆没有相应出版社的图书！\n");
    else bookView(pname_head);
    return ;
}
void bookSearchTime()
{
    struct bookInfo*current=head->next;
    struct bookInfo*newNode=NULL;
    struct bookInfo*btime_head=(struct bookInfo*)malloc(sizeof(bookInfo));
    memset(btime_head,0,sizeof(bookInfo));
    struct bookInfo*tail=btime_head;
    char btime[20];
    system("cls");
    printf("请输入待查找书籍的出版时间：\n");
    gets(btime);
    fflush(stdin);
    while(current!=NULL)
    {
        if(istime(current,btime))
        {
            newNode=(struct bookInfo*)malloc(sizeof(bookInfo));
            memset(newNode,0,sizeof(bookInfo));
            bookInfocpy(current,newNode);
            while(tail->next!=NULL)
                tail=tail->next;
            tail->next=newNode;
            newNode->next=NULL;
        }
        current=current->next;
    }
    if(btime_head->next==NULL)
        printf("图书馆没有相应出版时间的图书，扩大出版时间试试！\n");
    else bookView(btime_head);
    return ;
}



struct readerInfo*readerGetnode()
{
    struct  readerInfo*current=(struct  readerInfo*)malloc(sizeof(struct  readerInfo));
    memset(current,0,sizeof( readerInfo));
    printf("请输入读者编号：\n");
    scanf("%d",&current->no);
    fflush(stdin);
    printf("请输入读者姓名：\n");
    gets(current->readerName);
    fflush(stdin);
    printf("请输入借阅号：\n");
    gets(current->id);
    fflush(stdin);
    printf("请输入读者密码：\n");
    gets(current->password);
    fflush(stdin);
    printf("请输入读者最大借阅额度：\n");
    scanf("%d",&current->maxAmount);
    fflush(stdin);
    printf("请输入读者已借数量：\n");
    scanf("%d",&current->keepAmount);
    fflush(stdin);
    return current;
}
void readerInitialSave()
{
    FILE *fp;
    readerFree();
    rhead->next=NULL;
    char judege[20];
    struct readerInfo*tail=rhead;
    struct readerInfo*current=NULL;
    if ((fp = fopen("readerinformation.txt", "w+"))==NULL)
	{
		printf("不能打开读者信息文件夹！");
		return ;
	}
	do{
        system("cls");
        current=readerGetnode();
        while(tail->next!=NULL&&(tail->next->no)<(current->no))
            tail=tail->next;
        tail->next=current;
        current->next=NULL;
        printf("按任意键继续初始化读者信息，输入quit退出初始化！\n");
        gets(judege);
        fflush(stdin);
   }while(strcmp(judege,"quit")!=0);
   printf("图书信息初始完成！\n");
   return ;
}
void readerSave(struct readerInfo*rhead)
{
    FILE*fp;
    struct readerInfo*current=rhead->next;
    if ((fp = fopen("readerinformation.txt", "w"))==NULL)
	{
		printf("不能打开读者信息文件夹！");
		return ;
	}
    while(current!=NULL)
    {
        fwrite(current,sizeof(readerInfo),1,fp);
        current=current->next;
    }
    fclose(fp);
    printf("读者信息保存成功！\n");
}
void downloadReader()
{
    struct readerInfo*tail=rhead;
    struct readerInfo*current=NULL;
    FILE*fp;
    if ((fp = fopen("readerinformation.txt", "r+"))==NULL)
	{
		printf("不能打开读者信息文件夹！");
	}
	while(!feof(fp))
    {
        current=(struct readerInfo*)malloc(sizeof(readerInfo));
        memset(current,0,sizeof(readerInfo));
        if(fread(current,sizeof(readerInfo),1,fp)!=0)
        {
            tail->next=current;
            tail=current;
        }
        else free(current);
    }
    fclose(fp);
    return ;
}
void readerInsert()
{
    struct readerInfo*Newnode=NULL;
    struct readerInfo*current=rhead;
    system("cls");
    printf("请输入插入读者的相关信息\n\n");
    Newnode=readerGetnode();
    while(current->next!=NULL&&(current->next->no)<(Newnode->no))
        current=current->next;
    Newnode->next=current->next;
    current->next=Newnode;
    printf("添加成功！\n");
}
void readerDelete()
{
    struct readerInfo*current=rhead;
    struct readerInfo*temp=NULL;
    char readerName[20],id[20],judege[20];
    system("cls");
    printf("请输入待删除读者借阅号：\n");
    gets(id);
    fflush(stdin);
    while(current->next!=NULL)
    {
        if(strcmp(current->next->id,id)==0)
            break;
        current=current->next;
    }
    printf("您要删除的读者信息为：\n");
    readerModifyView(current->next);
    printf("按任意键删除，输入quit退出!\n");
    gets(judege);
    fflush(stdin);
    if(strcmp(judege,"quit")==0)
     {
        readeradminMenu();
        return;
     }
    temp=current->next;
    current->next=temp->next;
    free(temp);
    printf("删除成功！\n");
    return ;
}
struct readerInfo*readerSearch(char*readerID)
{
    struct readerInfo*current=rhead->next;
    while(current!=NULL)
    {
        if(strcmp(current->id,readerID)==0)
            return current;
        current=current->next;
    }
    return NULL;
}
void readerModifyMenu()
{
    printf("\t\t\t\t\t1.修改编号\n");
    printf("\t\t\t\t\t2.修改借阅号\n");
    printf("\t\t\t\t\t3.修改读者名\n");
    printf("\t\t\t\t\t4.修改最大借阅量\n");
    printf("\t\t\t\t\t5.修改已借量\n");
    printf("\t\t\t\t\t6.修改密码\n\n");
}
void readerModify()
{
    struct readerInfo*current=NULL;
    int choice;
    char id[20],judege[20],judege2[20];
    system("cls");
    printf("请输入待修改读者的借阅号：\n");
    gets(id);
    fflush(stdin);
    system("cls");
    current=readerSearch(id);
    if(current==NULL)
        printf("没有相应借阅号的读者！\n");
    else{
        printf("待修改的读者信息为：\n");
        readerModifyView(current);
        printf("按任意键继续修改，输入quit返回！\n");
        gets(judege2);
        fflush(stdin);
        if(strcmp(judege2,"quit")==0)
            return ;
        do{
            system("cls");
            readerModifyMenu();
            printf("请选择：\n");
            scanf("%d",&choice);
            fflush(stdin);
            switch(choice)
            {
            case 1:
                printf("请输入修改后的编号：\n");
                scanf("%d",&current->no);
                fflush(stdin);
                break;
            case 2:
                printf("请输入修改后的借阅号：\n");
                scanf("%s",&current->id);
                fflush(stdin);
                break;
            case 3:
                printf("请输入修改后的读者名：\n");
                scanf("%s",&current->readerName);
                fflush(stdin);
                break;
            case 4:
                printf("请输入修改后的最大借阅量：\n");
                scanf("%d",&current->maxAmount);
                fflush(stdin);
                break;
            case 5:
                printf("请输入修改后的已借量：\n");
                scanf("%d",&current->keepAmount);
                fflush(stdin);
                break;
            case 6:
                printf("请输入修改后的密码：\n");
                scanf("%s",&current->password);
                fflush(stdin);
                break;
            default:
            printf("没有相应的选项！\n");
            }
            printf("修改后的信息为；\n");
            readerModifyView(current);
            printf("按任意键继续修改当前读者信息，输入quit退出修改！");
            gets(judege);
            fflush(stdin);
        }while(strcmp(judege,"quit")!=0);
        printf("修改完成！\n");
    }
}
void readerView()
{
    system("cls");
    struct readerInfo*current=rhead->next;
    printf("编号              借阅号            姓名         最大借阅量         已借量          密码\n\n");
    while(current!=NULL)
    {
        printf("%-17d %-17s %-16s %-17d %-12d %-15s\n",current->no,
               current->id,current->readerName,current->maxAmount,current->keepAmount,current->password);
        current=current->next;
    }
}
void readerModifyView(struct readerInfo*current)
{
    if(current!=NULL)
    {
        printf("编号              借阅号            姓名         最大借阅量         已借量          密码\n\n");
        printf("%-17d %-17s %-16s %-17d %-12d %-15s\n",current->no,
                   current->id,current->readerName,current->maxAmount,current->keepAmount,current->password);
    }
}
/*********************************************************/
//借书信息的加载，通过书籍名，读者名，作者名查询借书信息
void borrowView(struct borrowInfo*head)
{
    struct borrowInfo*current=head->next;
    if(current!=NULL)
        printf("书名\t\t作者\t\t出版社\t\t读者名\t\t借阅号\t\t借阅时间\t状态\n\n");
    else printf("没有相关的借书信息！\n");
    while(current!=NULL)
    {
        printf("%-16s%-16s%-16s%-16s%-16s%-5d%-2d%-9d%-10s\n",current->bookName,current->writer,
               current->press,current->readerName,current->readerId,current->borrowTime.year,current->borrowTime.month,current->borrowTime.day,current->sign);
        current=current->next;
    }
}
void downloadBorrowInfo()
{
    FILE*fp;
    struct borrowInfo*tail=bohead;
    struct borrowInfo*current=NULL;
    if((fp=fopen("borrowInformation.txt","r+"))==NULL)
    {
        printf("借书信息文件夹打开失败！\n");
    }
    else
    {
        while(!feof(fp))
        {
            current=(struct borrowInfo*)malloc(sizeof(borrowInfo));
            memset(current,0,sizeof(borrowInfo));
            if(fread(current,sizeof(borrowInfo),1,fp)!=0)
            {
                 tail->next=current;
                 tail=current;
            }
            else free(current);
        }
    }
    fclose(fp);
    return;
}
void borrowSave(struct borrowInfo*bohead)
{
    FILE*fp;
    struct borrowInfo*current=bohead->next;
    fp=fopen("borrowInformation.txt","w+");
    while(current!=NULL)
    {
        fwrite(current,sizeof(borrowInfo),1,fp);
        current=current->next;
    }
    printf("借书信息保存成功！\n");
    fclose(fp);
}
void borrowSearch(int type,char*info)
{
    struct borrowInfo*current=bohead->next;
    struct borrowInfo*temphead=NULL;
    temphead=(struct borrowInfo*)malloc(sizeof(borrowInfo));
    struct borrowInfo*tail=temphead;
    memset(temphead,0,sizeof(borrowInfo));
    switch(type)
    {
    case 1:
        while(current!=NULL)
        {
            if(strcmp(current->bookName,info)==0)
                tail->next=current,
                tail=tail->next;
            current=current->next;
        }
       break;
    case 2:
        while(current!=NULL)
       {
            if(strcmp(current->readerId,info)==0)
                tail->next=current,
                tail=tail->next;
            current=current->next;
       }
       break;
    case 3:
        while(current!=NULL)
        {
            if(strcmp(current->writer,info)==0)
                tail->next=current,
                tail=tail->next;
            current=current->next;
        }
        break;
    }
    //tail->next=NULL;
    borrowView(temphead);
    free(temphead);
}
struct borrowInfo*borrowSearchUB(char*stuid,char*bookisbn)//找到读者唯一的借阅信息
{
   struct borrowInfo*current=bohead->next;
   while(current!=NULL)
   {
       if(strcmp(current->readerId,stuid)==0&&strcmp(current->isbn,bookisbn)==0)
        return current;
       current=current->next;
   }
   return NULL;
}
void borrowSearchBook()
{
    system("cls");
    char bookName[20];
    printf("请输入待查询的图书名字:\n");
    gets(bookName);
    fflush(stdin);
    borrowSearch(1,bookName);
}
void borrowSearchReader()
{
    system("cls");
    char readerID[20];
    printf("请输入待查询读者借阅证：\n");
    gets(readerID);
    fflush(stdin);
    borrowSearch(2,readerID);
}
void borrowSearchWriter()
{
    system("cls");
    char writer[20];
    printf("请输入待查询作者名：\n");
    gets(writer);
    fflush(stdin);
    borrowSearch(3,writer);
}

//还书信息的加载，通过不同的方法查询还书信息
void returnModifyView(struct returnInfo*current)
{
    printf("书名\t\t作者\t\t出版社\t\t读者名\t\t借阅号\t\t还书时间\n\n");
    printf("%-16s%-16s%-16s%-16s%-16s%-5d%-2d%-2d\n",current->bookName,current->writer,
               current->press,current->readerName,current->readerId,
               current->returnTime.year,current->returnTime.month,current->returnTime.day);
}
void returnView(struct returnInfo*head)
{
    struct returnInfo*current=head->next;
    if(current!=NULL)
        printf("书名\t\t作者\t\t出版社\t\t读者名\t\t借阅号\t\t还书时间\n\n");
    else printf("没有相关的还书信息！\n");
    while(current!=NULL)
    {
        printf("%-16s%-16s%-16s%-16s%-16s%-5d%-2d%-2d\n",current->bookName,current->writer,
               current->press,current->readerName,current->readerId,
               current->returnTime.year,current->returnTime.month,current->returnTime.day);
        current=current->next;
    }
}
void downloadReturnInfo()
{
    FILE*fp=NULL;
    struct returnInfo*tail=rehead;
    struct returnInfo*current=NULL;
    if((fp=fopen("returnInformation.txt","r+"))==NULL)
    {
        printf("还书信息文件夹打开失败！\n");
    }
    else
    {
        while(!feof(fp))
        {
            current=(struct returnInfo*)malloc(sizeof(returnInfo));
            memset(current,0,sizeof(returnInfo));
            if(fread(current,sizeof(returnInfo),1,fp)!=0)
            {
                 tail->next=current;
                 tail=current;
            }
            else free(current);
        }
    }
    fclose(fp);
    return ;
}
void returnSave(struct returnInfo*rehead)
{
    FILE*fp;
    struct returnInfo*current=rehead->next;
    fp=fopen("returnInformation.txt","w+");
    while(current!=NULL)
    {
        fwrite(current,sizeof(returnInfo),1,fp);
        current=current->next;
    }
    printf("还书信息保存成功！\n");
    fclose(fp);
}
void returnSearch(int type,char*info)
{
    struct returnInfo*current=rehead->next;
    struct returnInfo*temphead=NULL;
    temphead=(struct returnInfo*)malloc(sizeof(returnInfo));
    struct returnInfo*tail=temphead;
    memset(temphead,0,sizeof(returnInfo));
    switch(type)
    {
    case 1:
        while(current!=NULL)
        {
            if(strcmp(current->bookName,info)==0)
             {
                tail->next=current;
                tail=tail->next;
             }
            current=current->next;
        }
       break;
    case 2:
        while(current!=NULL)
       {
            if(strcmp(current->readerId,info)==0)
            {
                tail->next=current;
                tail=tail->next;
            }
            current=current->next;
       }
       break;
    case 3:
        while(current!=NULL)
        {
            if(strcmp(current->writer,info)==0)
             {
                tail->next=current;
                tail=tail->next;
             }
            current=current->next;
        }
        break;
    }
    returnView(temphead);
    free(temphead);
}
void returnSearchBook()
{
    system("cls");
    char bookName[20];
    printf("请输入待查询的图书名字:\n");
    gets(bookName);
    fflush(stdin);
    returnSearch(1,bookName);
}
void returnSearchReader()
{
    system("cls");
    char readerID[20];
    printf("请输入待查询读者借阅证：\n");
    gets(readerID);
    fflush(stdin);
    returnSearch(2,readerID);
}
void returnSearchWriter()
{
    system("cls");
    char writer[20];
    printf("请输入待查询作者名：\n");
    gets(writer);
    fflush(stdin);
    returnSearch(3,writer);
}

//读者以及管理员登陆
int readerLogin()
{
    struct readerInfo*current=rhead->next;
    char passWord[20];
    system("cls");
    printf("\t\t\t\t\t\t请输入借阅号\n");
    gets(stuID);
    fflush(stdin);
    printf("\t\t\t\t\t\t请输入密码\n");
    gets(passWord);
    fflush(stdin);
    while(current!=NULL)
    {
        if(strcmp(current->id,stuID)==0)
        {
            stuNode=current;
            for(int i=0;i<3;i++)
            {
                if(strcmp(current->password,passWord)==0)
                    return 1;
                else
                {
                    printf("密码错误！");
                    printf("请重新输入！\n");
                    printf("\t\t\t\t\t\t请输入密码\n");
                    gets(passWord);
                    fflush(stdin);
                    if(i==1)
                        return 0;
                }
            }
        }
        current=current->next;
    }
    if(current==NULL)
        printf("图书系统没有该读者的信息！\n");
    return 0;
}
int adminLogin()
{
    char ID[20];
    char passWord[20];
    system("cls");
    printf("\t\t\t\t\t    欢迎登陆\n\n");
    printf("\t\t\t\t\t请输入管理员账号\n");
    gets(ID);
    fflush(stdin);
    printf("\t\t\t\t\t请输入管理员密码\n");
    gets(passWord);
    fflush(stdin);
    if(strcmp(adminID,ID)==0&&strcmp(adminPassword,passWord)==0)
        return 1;
    return 0;
}
//图书借阅和还书
void bookBorrow()
{

    char isbn[20];
    char bookName[20];
    char judege[20];
    struct tm*localTime=NULL;//获取 得到当前时间函数的返回值
    struct borrowInfo*temp=NULL;
    struct bookInfo*current=NULL;
    struct borrowInfo*tail=bohead;
    struct borrowInfo*newNode=(struct borrowInfo*)malloc(sizeof(borrowInfo));
    system("cls");
    printf("\t\t\t\t\t请输入图书编号\n");
    gets(isbn);
    fflush(stdin);
    current=bookSearch(isbn);
    if(current==NULL)
    {
        printf("图书不存在！\n");
        return ;
    }
    else
    {
        printf("您要借阅的图书信息为：\n");
        bookModifyView(current);
        temp=borrowSearchUB(stuID,isbn);
        if(temp!=NULL)
        {
            if(strcmp(temp->sign,"未还")==0)
            {
                printf("图书仍在借阅期，请还书后再借！\n");
                return ;
            }//一次只能借一本书，如果已经借了还未归还不能再借
        }
        if((current->amount>0)&&((stuNode->keepAmount)<(stuNode->maxAmount)))
        {
            printf("图书可借！按任意键借阅，输入quit退出！\n");
            gets(judege);
            fflush(stdin);
            if(strcmp(judege,"quit")==0)
            {
                readeruserMenu();
                return ;
            }
            (stuNode->keepAmount)++;
            (current->amount)--;
            //创建借书信息链表(字符串不可以直接赋值)
            strcpy(newNode->isbn,isbn);
            strcpy(newNode->bookName,current->bookName);
            strcpy(newNode->writer,current->writer);
            strcpy(newNode->press,current->press);
            strcpy(newNode->readerName,stuNode->readerName);
            strcpy(newNode->readerId,stuID);
            strcpy(newNode->sign,"未还");
            //时间信息写入节点
            localTime=timeGet();
            newNode->borrowTime.day=localTime->tm_mday;
            newNode->borrowTime.month=localTime->tm_mon+1;
            newNode->borrowTime.year=localTime->tm_year+1900;
            //将链表连接在信息链表上
            while(tail->next!=NULL)//第二次指针NULL没有了指向所以直接终止了
                tail=tail->next;
            tail->next=newNode;
            newNode->next=NULL;
            printf("借书成功！\n");
        }
        else {
            if(current->amount<=0)
                printf("库存不足！");
            if((stuNode->keepAmount)>=(stuNode->maxAmount))
                printf("已到达最大借阅量，请还书后重试！\n");
        }

    }
}
void bookReturn()
{
    char bookName[20];
    char isbn[20];
    struct tm*returnTime=NULL;
    struct returnInfo*tail=rehead;
    struct returnInfo*recurrent=rehead->next;
    struct returnInfo*newNode=(struct returnInfo*)malloc(sizeof(struct returnInfo));
    memset(newNode,0,sizeof(returnInfo));
    struct borrowInfo*bocurrent=bohead->next;
    struct bookInfo*bkcurrent=NULL;
    int days=0;
    system("cls");
    printf("请输入图书编号\n");
    gets(isbn);
    fflush(stdin);
    bkcurrent=bookSearch(isbn);//找到图书信息节点
    if(bkcurrent==NULL)
    {
        printf("没有找到相应书籍！\n");
        return ;
    }
    while(bocurrent!=NULL)//找到借阅信息节点
    {
        if(strcmp(bocurrent->readerId,stuID)==0&&strcmp(bocurrent->isbn,isbn)==0&&strcmp(bocurrent->sign,"未还")==0)
        {
            printf("您的借书信息为：\n");
            printf("书名\t\t作者\t\t出版社\t\t读者名\t\t借阅号\t\t借阅时间\t图书状态\n\n");
            printf("%-16s%-16s%-16s%-16s%-16s%-5d%-2d%-8d%-10s\n",bocurrent->bookName,bocurrent->writer,
                       bocurrent->press,bocurrent->readerName,bocurrent->readerId,bocurrent->borrowTime.year,bocurrent->borrowTime.month,bocurrent->borrowTime.day,bocurrent->sign);
            returnTime=timeGet();
            days=dayCount(bocurrent->borrowTime.year,bocurrent->borrowTime.month,bocurrent->borrowTime.day,
                              returnTime->tm_year+1900,returnTime->tm_mon+1,returnTime->tm_mday);
            if(days>MAXDAY)
            {
                printf("图书借阅超期，请到管理员处归还图书!");
                    return;
            }
            else  printf("图书可还！按任意键还书！\n");
            getchar();
            (bkcurrent->amount)++;//修改图书数量信息
            (stuNode->keepAmount)--;//修改读者信息
            strcpy(bocurrent->sign,"已还");
            strcpy(newNode->isbn,isbn);
            strcpy(newNode->bookName,bkcurrent->bookName);
            strcpy(newNode->writer,bkcurrent->writer);
            strcpy(newNode->press,bkcurrent->press);
            strcpy(newNode->readerName,stuNode->readerName);
            strcpy(newNode->readerId,stuID);
            //时间信息写入节点
            returnTime=timeGet();
            newNode->returnTime.day=returnTime->tm_mday;
            newNode->returnTime.month=returnTime->tm_mon+1;
            newNode->returnTime.year=returnTime->tm_year+1900;
            //将链表连接在信息链表上
            while(tail->next!=NULL)
                tail=tail->next;
            tail->next=newNode;
            newNode->next=NULL;//还书信息链表添加一条还书信息
            printf("还书成功！\n");
            return ;
        }
        bocurrent=bocurrent->next;
    }
    if(bocurrent==NULL)
        printf("系统没有相关借阅信息，无法还书！\n");
    free(newNode);
    return ;
}
void bookReturnAdmin()
{
    char bookName[20];
    char isbn[20];
    struct tm*returnTime=NULL;
    struct returnInfo*tail=rehead;
    struct returnInfo*recurrent=rehead->next;
    struct returnInfo*newNode=(struct returnInfo*)malloc(sizeof(struct returnInfo));
    memset(newNode,0,sizeof(returnInfo));
    struct borrowInfo*bocurrent=bohead->next;
    struct bookInfo*bkcurrent=NULL;
    int days=0;
    system("cls");
    printf("请输入图书编号\n");
    gets(isbn);
    fflush(stdin);
    bkcurrent=bookSearch(isbn);//找到图书信息节点
    if(bkcurrent==NULL)
    {
        printf("没有找到相应书籍！\n");
        return ;
    }
    while(bocurrent!=NULL)//找到借阅信息节点
    {
        if(strcmp(bocurrent->readerId,stuID)==0&&strcmp(bocurrent->isbn,isbn)==0)
        {
            printf("您的借书信息为：\n");
            printf("书名\t\t作者\t\t出版社\t\t读者名\t\t借阅号\t\t借阅时间\t\t图书状态\n\n");
            printf("%-16s%-16s%-16s%-16s%-16s%-5d%-2d%-2d%-10s\n",bocurrent->bookName,bocurrent->writer,
                       bocurrent->press,bocurrent->readerName,bocurrent->readerId,bocurrent->borrowTime.year,bocurrent->borrowTime.month,bocurrent->borrowTime.day,bocurrent->sign);
            returnTime=timeGet();
            days=dayCount(bocurrent->borrowTime.year,bocurrent->borrowTime.month,bocurrent->borrowTime.day,
                              returnTime->tm_year+1900,returnTime->tm_mon+1,returnTime->tm_mday);
            printf("应缴纳的逾期费用为：%lf\n",(days-MAXDAY)*rate);
            printf("图书可还！按任意键继续！\n");
            getchar();
            (bkcurrent->amount)++;//修改图书数量信息
            (stuNode->keepAmount)--;//修改读者信息
            strcpy(bocurrent->sign,"已还");
            strcpy(newNode->isbn,isbn);
            strcpy(newNode->bookName,bkcurrent->bookName);
            strcpy(newNode->writer,bkcurrent->writer);
            strcpy(newNode->press,bkcurrent->press);
            strcpy(newNode->readerName,stuNode->readerName);
            strcpy(newNode->readerId,stuID);
            //时间信息写入节点
            returnTime=timeGet();
            newNode->returnTime.day=returnTime->tm_mday;
            newNode->returnTime.month=returnTime->tm_mon+1;
            newNode->returnTime.year=returnTime->tm_year+1900;
            //将链表连接在信息链表上
            while(tail->next!=NULL)
                tail=tail->next;
            tail->next=newNode;
            newNode->next=NULL;                     //还书信息链表添加一条还书信息
            return ;
        }
        bocurrent=bocurrent->next;
    }
    if(bocurrent==NULL)
        printf("系统没有相关借阅信息，无法还书！\n");
    free(newNode);
    return ;
}
//所有链表的free
void bookFree()
{
    struct bookInfo*current=head->next;
    struct bookInfo*temp=NULL;
    while(current!=NULL)
    {
        temp=current;
        current=current->next;
        free(temp);
    }
}
void readerFree()
{
    struct readerInfo*current=rhead->next;
    struct readerInfo*temp=NULL;
    while(current!=NULL)
    {
        temp=current;
        current=current->next;
        free(temp);
    }
}
void borrowFree()
{
    struct borrowInfo*current=bohead->next;
    struct borrowInfo*temp=NULL;
    while(current!=NULL)
    {
        temp=current;
        current=current->next;
        free(temp);
    }
}
void returnFree()
{
    struct returnInfo*current=rehead->next;
    struct returnInfo*temp=NULL;
    while(current!=NULL)
    {
        temp=current;
        current=current->next;
        free(temp);
    }
}

void signUp()//获取一条读者信息，姓名 借阅号 读者密码
{
    char id[20];
    char password[20];
    char readerName[20];
    struct readerInfo*current=rhead;
    struct readerInfo*newNode=(struct readerInfo*)malloc(sizeof(readerInfo));
    memset(newNode,0,sizeof(readerInfo));
    printf("请输入您的姓名：\n");
    gets(newNode->readerName);
    fflush(stdin);
    printf("请输入您的借阅号：\n");
    gets(newNode->id);
    fflush(stdin);
    printf("请输入您的借阅密码：\n");
    gets(newNode->password);
    fflush(stdin);
    while(current->next!=NULL)
        current=current->next;
    if(current==rhead)
        newNode->no=0;
    else newNode->no=(current->no)+1;//错误：一开始使用了自增自减使得上一个的编号发生变化
    newNode->keepAmount=0;//而新的信息编号没变
    newNode->maxAmount=12;

    current->next=newNode;
    newNode->next=NULL;
    printf("注册成功！\n");
    return ;
}

void userModify()
{
    int choice;
    char id[20],judege[20],readerName[20],password[20];
    struct readerInfo*current=NULL;
    printf("请输入您的借阅号：\n");
    gets(id);
    fflush(stdin);
    current=readerSearch(id);
    if(current==NULL)
    {
        printf("读者不存在！\n");
        readeruserMenu();
        return ;
    }
    else
    {
       printf("待修改读者信息为：\n");
       readerModifyView(current);
       system("cls");
       printf("\t\t\t\t\t1.修改读者名\n");
       printf("\t\t\t\t\t2.修改密码\n\n");
       printf("\t\t\t\t\t0.返回主菜单\n");
       printf("请选择：\n");
       scanf("%d",&choice);
       fflush(stdin);
       switch(choice)
       {
       case 1:
        printf("输入修改后的读者名:\n");
        gets(readerName);
        fflush(stdin);
        strcpy(current->readerName,readerName);
        break;
       case 2:
        printf("输入修改后的读者密码：\n");
        gets(password);
        fflush(stdin);
        strcpy(current->password,password);
        break;
       case 0:
        mainMenu();
        return ;
       }
       printf("修改后的信息为：\n");
       readerModifyView(current);
    }

}
int main()
{
    memsetAll();
    downloadBook();
    downloadReader();
    downloadReturnInfo();
    downloadBorrowInfo();

    mainMenu();//mainMenu需要添加登陆函数
    bookSave(head);
    readerSave(rhead);
    borrowSave(bohead);
    returnSave(rehead);
    readerFree();
    bookFree();
    borrowFree();
    returnFree();
    free(head);
    free(rhead);
    free(bohead);
    free(rehead);
    printf("\n\n谢谢使用！\n");
}
