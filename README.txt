Setting up testing harness:
-Copy the files to your nachos repository.
-For eclipse: Create a new run configuration using JUnit 4.0 launcher. Set the base folder to the root of the project and the working directory (under the Arguments tab) to nachos/test/unittest
-Outside of eclipse: To be written.
-To create tests, subclass TestingHarness.java. Tests are created using the method enqueueJob:

@Test
public void testMethod() {
	enqueueJob(
		new Runnable() {
			public void run() {
				// Some testing code goes here
				KThread.currentThread.sleep(); // nachos code
				Assert.assertTrue(true);       // JUnit assertions
			}
		}
	);
}

To use a different scheduler have your test code override

protected static Class<? extends Scheduler> getScheduler() {
	return /*NameOfYourSchedulerClass.class;*/
}

Current Bugs:
-not possible to use multiple test Classes
-Submit bugs and/or fixes to alex.issayev@gmail.com

-------------------------Sample Test code:

package nachos.test.unittest;

import static org.junit.Assert.assertTrue;
import nachos.threads.Communicator;
import nachos.threads.KThread;
import nachos.threads.ThreadedKernel;

import org.junit.Test;

public class BasicRun extends TestHarness {

	@Test
	public void testRunNachos() {
		assertTrue(true);
	}

	@Test
	public void testCommunicator() {
		enqueueJob(new Runnable() {

			@Override
			public void run() {
				Communicator commu = new Communicator();
				Listener listener = new Listener(commu);
				KThread thread1 = new KThread(new Speaker(0xdeadbeef, commu))
						.setName("Speaker Thread");
				thread1.fork();
				listener.run();
				assertTrue("Incorrect Message recieved",
						0xdeadbeef == listener.getMessage());
			}

		});
	}

	private static class Listener implements Runnable {
		private int msg;
		private Communicator commu;
		private boolean hasRun;

		private Listener(Communicator commu) {
			this.commu = commu;
			this.hasRun = false;
		}

		public void run() {
			msg = commu.listen();
			hasRun = true;
		}

		private int getMessage() {
			assertTrue("Listener has not finished running", hasRun);
			return msg;
		}
	}

	private static class Speaker implements Runnable {
		private int msg;
		private Communicator commu;

		private Speaker(int msg, Communicator commu) {
			this.msg = msg;
			this.commu = commu;
		}

		public void run() {
			commu.speak(msg);
		}
	}
}
