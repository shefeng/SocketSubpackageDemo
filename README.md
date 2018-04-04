# SocketSubpackageDemo
这是一个用java编写的socket分包demo

代码如下：


import java.io.ByteArrayOutputStream;
import java.io.DataOutputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;

public class SocketServerDemo {

	private static ServerSocket ss;

	public static void main(String[] args) throws Exception {

		ss = new ServerSocket(12345);
		System.out.println("服务端已启动，正在监听12345端口...");

		while (true) {
			// 等待客户端
			Socket s = ss.accept();
			System.out.println("检测到客户端，准备数据接收...");

			// 读取本地的图片
			byte[] data = fileToBytes("/Users/shefeng/Documents/workspace-sts-3.9.1.RELEASE/TestSocket/src/timg.jpeg");

			// 将流输出给客户端
			DataOutputStream out = new DataOutputStream(s.getOutputStream());
			sendBytes(data, out);
		}
	}

	public static void sendBytes(byte[] data, DataOutputStream out) throws IOException {

		System.out.println("文件长度:" + (int) data.length);

		int packetType = 0;// 包类别 （根据自己的业务逻辑定义，非必须）
		int dataLen = data.length;// 整包数据大小
		int packageSize;// 包大小

		int remain = data.length;

		do {
			packageSize = (remain <= 4096 - 8) ? remain : (4096 - 8);

			/*
			 * The Packet Header Format：
			 *  |    包类型    |    包长度    |    包数据    |
			 *  |-- 4 bytes --|-- 4 bytes --|-- ? bytes --|
			 */

			// 每个包最大4096 = 8 + 4088 （8为头的长度，4088为分包数据）
			byte[] packet = new byte[4096];

			packet[0] = (byte) (packetType & 0xFF);
			packet[1] = (byte) ((packetType >> 8) & 0xFF);
			packet[2] = (byte) ((packetType >> 16) & 0xFF);
			packet[3] = (byte) ((packetType >> 24) & 0xFF);

			packet[4] = (byte) (dataLen & 0xFF);
			packet[5] = (byte) ((dataLen >> 8) & 0xFF);
			packet[6] = (byte) ((dataLen >> 16) & 0xFF);
			packet[7] = (byte) ((dataLen >> 24) & 0xFF);

			//将data中从（dataLen - remain）开始，长度为packageSize的数据拷贝到packet中 （即每一个包）
			System.arraycopy(data, dataLen - remain, packet, 8, packageSize);

			//输出给客户端
			out.write(packet, 0, 4096);

			out.flush();

			remain -= packageSize;

		} while (remain > 0);
	}

	// file2byte
	public static byte[] fileToBytes(String filePath) {
		byte[] buffer = null;
		File file = new File(filePath);

		FileInputStream fis = null;
		ByteArrayOutputStream bos = null;

		try {
			fis = new FileInputStream(file);
			bos = new ByteArrayOutputStream();

			byte[] b = new byte[1024];

			int n;

			while ((n = fis.read(b)) != -1) {
				bos.write(b, 0, n);
			}

			buffer = bos.toByteArray();
		} catch (FileNotFoundException ex) {

		} catch (IOException ex) {

		} finally {
			try {
				if (null != bos) {
					bos.close();
				}
			} catch (IOException ex) {

			} finally {
				try {
					if (null != fis) {
						fis.close();
					}
				} catch (IOException ex) {

				}
			}
		}

		return buffer;
	}
}
