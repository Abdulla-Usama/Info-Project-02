import java.util.*;

class MySemaphore {
    private int permits;

    public MySemaphore(int permits) {
        this.permits = permits;
    }

    public synchronized void acquire() {
        while (permits <= 0) {
            try {
                wait();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
        permits--;
    }

    public synchronized void release() {
        permits++;
        notify();
    }
}

public class Main {
    static int stock = 25;
    static int served = 0;
    static int rejected = 0;
    static MySemaphore mutex = new MySemaphore(1);

    static class Fan extends Thread {
        int id;

        public Fan(int id) {
            this.id = id;
        }

        @Override
        public void run() {
            for (int i = 0; i < 6; i++) {
                mutex.acquire();
                try {
                    if (stock > 0) {
                        stock--;
                        served++;
                    } else {
                        rejected++;
                    }
                } finally {
                    mutex.release();
                }
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Fan[] fans = new Fan[5];

        for (int i = 0; i < 5; i++) {
            fans[i] = new Fan(i + 1);
            fans[i].start();
        }

        for (int i = 0; i < 5; i++) {
            fans[i].join();
        }

        System.out.println("Tickets sold: " + served);
        System.out.println("Fans rejected: " + rejected);
        System.out.println("Tickets left: " + stock);
    }
}
