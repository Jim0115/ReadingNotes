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
        NSLog(@"File downloaded to: %@", filePath);
      }];
      
      [downloadTask resume];
使用AFNetworking下载一张图片。输出结果为

    File downloaded to: file:///var/mobile/Containers/Data/Application/A2C44130-3EF7-45DF-BE49-639A22C408AD/Documents/bd_logo1_31bdc765.png
    
AFNetworking将图片保存在用户路径下，如存在本地文件则优先使用本地文件，但如果无网络则不能将本地图片离线使用。在检查是否存在文件时只是简单的文件名比对，并不会将已过期的文件替换为新文件。
    
---

      NSURLSessionDownloadTask* downloadTask = [[NSURLSession sharedSession] downloadTaskWithRequest:request completionHandler:^(NSURL * _Nullable location, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        [[NSOperationQueue mainQueue] addOperationWithBlock:^{
          self.imageView.image = [UIImage imageWithData:[NSData dataWithContentsOfURL:location]];
          NSLog(@"File downloaded to: %@", location);
        }];
      }];
使用NSURLSession下载一张图片。输出信息为：

    file:///private/var/mobile/Containers/Data/Application/4D3F7F8F-FB5C-4A2E-A930-559BB1E3AAC3/tmp/CFNetworkDownload_7qI10c.tmp
    
且每次均生产不同的临时文件。
