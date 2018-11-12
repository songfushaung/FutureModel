# FutureModel
线程callable的future模式手写
public interface Data {
	public abstract String getRequest();
}

public class FurureData implements Data {

	public volatile static boolean ISFLAG = false;
	private RealData realData;

	public synchronized void setRealData(RealData realData) {
		// 如果已经获取到结果，直接返回
		if (ISFLAG) {
			return;
		}
		// 如果没有获取到数据,传递真是对象
		this.realData = realData;
		ISFLAG = true;
		// 进行通知
		notify();
	}

	@Override
	public synchronized String getRequest() {
		while (!ISFLAG) {
			try {
				wait();
			} catch (Exception e) {

			}
		}
		// 获取到数据,直接返回
		return realData.getRequest();
	}

}


public class RealData implements Data {
	private String result;

	public RealData(String data) {
		System.out.println("正在使用data:" + data + "网络请求数据,耗时操作需要等待.");
		try {
			Thread.sleep(3000);
		} catch (Exception e) {

		}
		System.out.println("操作完毕,获取结果...");
		result = "余胜军";
	}

	@Override
	public String getRequest() {
		return result;
	}
  
  public class FutureClient {

	public Data request(String queryStr) {
		FurureData furureData = new FurureData();
		new Thread(new Runnable() {

			@Override
			public void run() {
				RealData realData = new RealData(queryStr);
				furureData.setRealData(realData);
			}
		}).start();
		return furureData;

	}

}

public class Main {

	public static void main(String[] args) {
		FutureClient futureClient = new FutureClient();
		Data request = futureClient.request("请求参数.");
		System.out.println("请求发送成功!");
		System.out.println("执行其他任务...");
		String result = request.getRequest();
		System.out.println("获取到结果..." + result);
	}

}
