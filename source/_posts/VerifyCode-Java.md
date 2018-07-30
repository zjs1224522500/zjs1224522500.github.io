---
title: '验证码Java实现'
date: 2017-10-29 21:28:52
tags: JavaWeb
---
# 验证码Java实现
![image](http://oxp7d1tae.bkt.clouddn.com/%E9%AA%8C%E8%AF%81%E7%A0%81.png)
<!-- more -->
## 1、`Servlet`实现验证码
### 实现步骤:
- 1、服务端随机生成验证码：利用`java.awt`中的相关类，譬如`Graphics` **（核心）**,`BufferedImage`
- 2、利用`Servlet`进行`Client`和`Server`的交互，进行验证码的获取，以及表单的客户端验证码输入和服务端存储的记录进行验证。

### 核心代码：（验证码的生成）
```Java
public class ImageServlet extends HttpServlet {

	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp)
			throws ServletException, IOException {
		//初始化图片类型
		BufferedImage bufferedImage = new BufferedImage(68, 22, BufferedImage.TYPE_INT_RGB);
		//进行制图
		Graphics graphics = bufferedImage.getGraphics();
		//RGB设置颜色并填充对应图像坐标像素点
		Color color = new Color(200, 150, 255);
		graphics.setColor(color);
		graphics.fillRect(0, 0, 68, 22);
		char[] chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789".toCharArray();
		Random random = new Random();
		int length = chars.length, index;

		//循环随机生成验证码
		StringBuffer stringBuffer = new StringBuffer();
		for (int i = 0; i < 4; i++) {
			index = random.nextInt(length);
			//设置验证码的不同颜色并记录验证码
			graphics.setColor(
					new Color(random.nextInt(88), random.nextInt(188), random.nextInt(255)));
			graphics.drawString(chars[index] + "", (i * 15) + 3, 18);
			stringBuffer.append(chars[index]);
		}
		//将验证码的记录值存储在session中
		req.getSession().setAttribute("picCode", stringBuffer.toString());
		//将生成的图片写入Response的输出流中
		ImageIO.write(bufferedImage, "JPG", resp.getOutputStream());
	}
}
```


## 2、插件实现验证码：
- 常见验证码组件 `Jcaptcha`和`Kaptcha`
---
敬请期待 :)

## 3、Java图片验证码
敬请期待 :)