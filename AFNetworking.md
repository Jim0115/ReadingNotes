# AFNetworking
一个轻量级的网络框架。
## Download Task

    NSURLSessionConfiguration* configuration = [NSURLSessionConfiguration defaultSessionConfiguration];
      AFURLSessionManager* manager = [[AFURLSessionManager alloc] initWithSessionConfiguration:configuration];
      
      NSURL* url = [NSURL URLWithString:@"https://ss0.bdstatic.com/5aV1bjqh_Q23odCf/static/superman/img/logo/bd_logo1_31bdc765.png" relativeToURL:nil];
      NSURLRequest* request = [NSURLRequest requestWithURL:url];
      
      NSURLSessionDownloadTask *downloadTask = [manager downloadTaskWithRequest:request progress:nil destination:^NSURL *(NSURL *targetPath, NSURLResponse *response) {
      NSURL *documentsDirectoryURL = [[NSFileManager defaultManager] URLForDirectory:NSDocumentDirectory inDomain:NSUserDomainMask appropriateForURL:nil create:NO error:nil];
        return [documentsDirectoryURL URLByAppendingPathComponent:[response suggestedFilename]];
      } completionHandler:^(NSURLResponse *response, NSURL *filePath, NSError *error) {
        NSLog(@"%@", [NSOperationQueue currentQueue]);
        self.imageView.image = [UIImage imageWithData:[NSData dataWithContentsOfURL:filePath]];
        self.imageView.contentMode = UIViewContentModeTop;
        NSLog(@"File downloaded to: %@", filePath);
      }];
      
      [downloadTask resume];
使用AFNetworking下载一张图片。

      NSURLSessionDownloadTask* downloadTask = [[NSURLSession sharedSession] downloadTaskWithRequest:request completionHandler:^(NSURL * _Nullable location, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        [[NSOperationQueue mainQueue] addOperationWithBlock:^{
          self.imageView.image = [UIImage imageWithData:[NSData dataWithContentsOfURL:location]];
        }];
      }];
使用NSURLSession下载一张图片。