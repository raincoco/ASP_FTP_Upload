# ASP_FTP_Upload
ASP.NET的FTP上传
···
public class FtpWeb
{
    public static string ftpHost;
    public static string ftpUserID;
    public static string ftpPassword;

    /// <param name="ftpURI">ftp上的路径</param>
    /// <param name="filename">ftp上的文件名</param>
    /// <param name="fileLength">文件大小</param>
    /// <param name="localStream">本地文件流</param>
    /// <param name="errorMsg">报错信息</param>
    
    public static bool Upload(string ftpURI, string filename, int fileLength, Stream localStream, out string errorMsg)
    {
        errorMsg = "";
        Stream fileStream = null;       // 构造一个本地文件流fileStream
        Stream requestStream = null;    // 构造一个ftp文件流requestStream
                                      
        try
        {
            fileStream = localStream; // fileStream本地文件流获取本地文件流

            /* 连接FTP服务器 */
            Uri uri = new Uri(ftpHost + ftpURI + "/" + filename);                       // 构造资源标识符uri标识ftp完整路径
            FtpWebRequest uploadRequest = (FtpWebRequest)WebRequest.Create(uri);        // FtpWebRequest创建FTP连接，创建实例uploadRequest  
            uploadRequest.Method = WebRequestMethods.Ftp.UploadFile;                    // 将FtpWebRequest属性设置为上传文件  
            uploadRequest.Credentials = new NetworkCredential(ftpUserID, ftpPassword);  // 认证FTP用户名密码  
            requestStream = uploadRequest.GetRequestStream();                           // 存储要由当前请求发送到服务器的数据，返回Stream

            /* 文件流操作 */
            int buffLength = 2048; // 2KB缓冲区  
            byte[] buff = new byte[buffLength]; // byte[]八位无符号数组

            int contentLen; // 数据大小（总字节数）
            contentLen = fileStream.Read(buff, 0, buffLength); // 将本地文件流中的数据读入buff缓冲区中。返回读入缓冲区的总字节数。

            // 读取本地文件流数据到缓冲区
            while (contentLen != 0) // contentLen为0，则表示本地文件流数据已经全部读完。
            //以读入缓冲区的字节总数contentLen为0作为判断条件，是因为每次读取的字节数可能小于缓冲区的大小。
            
            {
                requestStream.Write(buff, 0, contentLen); // 将buff缓冲区内的数据写入ftp文件流
                contentLen = fileStream.Read(buff, 0, buffLength); // 继续从本地文件流向缓冲区读取数据，返回读取字节数。
            }

            /* 关闭文件流释放资源 */
            requestStream.Close();  // 关闭ftp文件流，释放相关资源（如套接字和文件句柄）
            fileStream.Close();     // 关闭本地文件流，释放相关资源（如套接字和文件句柄）

            return true;
        }
        catch (Exception ex) // 出现异常，捕获异常对象ex
        {
            errorMsg = ex.Message;
            return false;
        }
        finally
        {
            if (fileStream != null) fileStream.Close();
            if (requestStream != null) requestStream.Close();
        }
    }


}
···
