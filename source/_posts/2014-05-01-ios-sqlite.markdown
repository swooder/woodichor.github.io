---
layout: post
title: "ios sqlite3使用"
date: 2014-05-01 00:20:18 +0800
comments: true
categories: IOS
---

#####iOS中使用
在 iOS 中 sqlite3 库是一套纯 C 的接口，因此很方便地就可以在 obj-c 源码中无痕使用它
首先，需要在Frameworks中加入所需的库Library   libsqlite3.0.dylib

这样可以导入头文件了

```
#import "ViewController.h"
#import "sqlite3.h"
#define kDatabaseName @"city.db"
@interface ViewController (){
      sqlite3 * database;
}
@end
```

####初始化数据库

```
-(BOOL)initializeDb {
    NSLog(@"initializeDB");
    // look to see if DB is in known location (~/Documents/$DATABASE_FILE_NAME)
    //START:code.DatabaseShoppingList.findDocumentsDirectory
    NSArray *searchPaths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
    NSString *documentFolderPath = [searchPaths objectAtIndex:0];
    //查看文件目录
    NSLog(@"%@", documentFolderPath);
    self.databaseFilePath = [documentFolderPath stringByAppendingPathComponent:@"city.db"];
    
    if(![[NSFileManager defaultManager] fileExistsAtPath:self.databaseFilePath]){
        //didnit find db, need to copy
        NSString *backupDbPath = [[NSBundle mainBundle] pathForResource:@"city" ofType:@"db"];
        
        if(backupDbPath == nil) {
            //couldn't find backup db to copy
            return NO;
        }else {
            BOOL copiedBackupDb = [[NSFileManager defaultManager] copyItemAtPath:backupDbPath toPath:self.databaseFilePath error:nil];
            if(!copiedBackupDb){
                return NO;
            }
        }
        
    }
    return TRUE;
}
```

####创建表

```
-(BOOL) createTable{
    char *sql = "CREATE TABLE city (id integer primary key, cid integer, cityName text)";
    sqlite3_stmt *statment;
    char *errorMsg;
    if(sqlite3_exec(database, sql, NULL, NULL, &errorMsg) != SQLITE_OK){
        NSLog(@"Error: create table failed %s", sqlite3_errmsg(database));
        return NO;
    }
    
    int success = sqlite3_finalize(statment);
    if(success != SQLITE_DONE){
        NSLog(@"failed to dehydate: %s", sqlite3_errmsg(database));
        return NO;
    }
    return YES;
}
```

####向表中插入记录

```
-(BOOL) insertOne{
    char *errorMsg;
    const char *sql = "insert into city (cid, cityName) values(101, '上海')";
    if(sqlite3_exec(database, sql, NULL, NULL, &errorMsg) == SQLITE_OK){
        NSLog(@"Insert ok");
        return YES;
    }else{
        NSLog(@"error: %s",errorMsg);
        sqlite3_free(errorMsg);
        return  NO;
    }
}
```

####查询数据库

```
-(void) showCitys{
    const char * sql = "select * from city";
    sqlite3_stmt *statment;
    if(sqlite3_prepare_v2(database, sql, -1, &statment, nil) == SQLITE_OK) {
        NSLog(@"select ok.");
    }
    while(sqlite3_step(statment) == SQLITE_ROW) {
        int id = sqlite3_column_int(statment, 1);
        //char *name = (char *) sqlite3_column_text(statment, 2);
        //直接用的char类型来中文会有乱码，使用NSString代替
        NSString *name=[[NSString alloc] initWithCString:(char *)sqlite3_column_text(statment , 2) encoding:NSUTF8StringEncoding];
        NSLog(@"row >>id %i, name %@", id, name);
    }
    
    sqlite3_finalize(statment);
    
}
```

####关闭数据库

```
- (void)viewDidUnload {   
    
   sqlite3_close(database);   

} 
```




