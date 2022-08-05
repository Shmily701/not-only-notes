
在Activity#onActivityResult(int requestCode, int resultCode, Intent data)

data.getData() 返回值为uri 将uri转换为byte数组，方式如下

```
Uri data = result.getData();   
ByteArrayOutputStream baos = new ByteArrayOutputStream();  
FileInputStream fis;  
try {  
    fis = new FileInputStream(new File(data.getPath()));  
    byte[] buf = new byte[1024];  
    int n;  
    while (-1 != (n = fis.read(buf)))  
        baos.write(buf, 0, n);  
} catch (Exception e) {  
    e.printStackTrace();  
}  
byte[] bbytes = baos.toByteArray();

```

这里涉及到camera取景获取到的图像资源信息。todo