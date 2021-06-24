package grsd_p01_soap_client;

import java.io.BufferedWriter;
import java.io.FileWriter;
import java.io.IOException;
import java.io.UnsupportedEncodingException;
import java.util.Random;

// Correr na VM2.
public class GRSD_P01_SOAP_Client {

    /**
     * @param args
     * @throws java.io.IOException
     */
    public static void main(String[] args) throws IOException {

        final String CSV_HEADER = ("SOAP" + "," + "Local" + "\n"); // Cabeçalho do ficheiro output. (Opcional)
        final String CSV_FILENAME = ("P01_log"); // Nome do ficheiro output.
        final int SAMPLE_NUMBER = 20;
        final int TIMEOUT = 3000; // Em milisegundos.
        final int START_PACKET_SIZE = 1; // Em bytes.
        final int MAXIMUM_PACKET_SIZE = (int) 1e9; // Em bytes.

        FileWriter logfile = new FileWriter(CSV_FILENAME + ".csv");
        try (BufferedWriter logwrite = new BufferedWriter(logfile)) {
            logwrite.write(CSV_HEADER);
            logwrite.write("Packet size (B)" + "," + "Time (ms)" + "," + "Time average (ms)" + "\n");

            long tt = 0;
            double t_average = 0;
            int i;
            for (int PacketSize = START_PACKET_SIZE; tt < TIMEOUT && PacketSize < MAXIMUM_PACKET_SIZE; PacketSize = PacketSize * 10) {
                if (PacketSize < 1e3) {
                    System.out.println("\nPacket size: " + PacketSize + " B\n");
                } else if (PacketSize < 1e6) {
                    System.out.println("\nPacket size: " + PacketSize / 1e3 + " KB\n");
                } else {
                    System.out.println("\nPacket size: " + PacketSize / 1e6 + " MB\n");
                }
                for (i = 0; i < SAMPLE_NUMBER && tt < TIMEOUT; i++) {
                    String Packet = randomStringGenerator(PacketSize);
                    //debugPacketSize(Packet, PacketSize);
                    long t_start = System.currentTimeMillis();
                    hello(Packet);
                    long t_finish = System.currentTimeMillis();
                    tt = t_finish - t_start;
                    t_average = t_average + tt;
                    System.out.println("Sample index: " + (i + 1) + "/" + SAMPLE_NUMBER + "\n" + "Time: " + tt + " ms\n");
                    logwrite.write(PacketSize + "," + tt + "\n");
                }
                if (i == SAMPLE_NUMBER) { // Esta condição só se verifica se todas as amostras tiverem sido recolhidas durante o ciclo "for" anterior.
                    t_average = t_average / SAMPLE_NUMBER;
                    System.out.println("Time average: " + t_average + " ms\n");
                    logwrite.write(PacketSize + ",," + t_average + "\n");
                    t_average = 0;
                }
            }
            logwrite.flush();
            logwrite.close();
        }
        System.out.println("\nEND");
    }

    private static void hello(String pac) {
        grsd_p01_soap_server.Hello_Service service = new grsd_p01_soap_server.Hello_Service();
        grsd_p01_soap_server.Hello port = service.getHelloPort();
        port.hello(pac);
    }

    private static String randomStringGenerator(int PacketSize) {
        int StringLength = (int) Math.ceil((float) PacketSize / 2);
        Random random = new Random();
        StringBuilder sbRandString = new StringBuilder(StringLength);

        //String allowedChars = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";
        for (int i = 0; i < StringLength; i++) {
            //sbRandString.append(allowedChars.charAt(random.nextInt(allowedChars.length())));
            char RandChar = (char) random.nextInt();

            // Filtrar caracteres ilegais em XML.
            while ((RandChar < 0x9)
                    || ((RandChar > 0x9) && (RandChar < 0xA))
                    || ((RandChar > 0xA) && (RandChar < 0xD))
                    || ((RandChar > 0xD) && (RandChar < 0x20))
                    || ((RandChar > 0xD7FF) && (RandChar < 0xE000))
                    || ((RandChar > 0xFFFD) && (RandChar < 0x10000))
                    || (RandChar > 0x10FFFF)) {
                RandChar = (char) random.nextInt();
            }
            /*
            // Filtrar caracteres ilegais em XML (alternativa).
            while (!((RandChar == 0x9)
                    || (RandChar == 0xA)
                    || (RandChar == 0xD)
                    || ((RandChar >= 0x20) && (RandChar <= 0xD7FF))
                    || ((RandChar >= 0xE000) && (RandChar <= 0xFFFD))
                    || ((RandChar >= 0x10000) && (RandChar <= 0x10FFFF)))) {
                RandChar = (char) random.nextInt();
            }
             */
            sbRandString.append(RandChar);
        }

        return sbRandString.toString();
    }

    private static void debugPacketSize(String Packet, int PacketSize) throws UnsupportedEncodingException {
        // Este método de debugging mede o tamanho do pacote gerado, que é do tipo String, em bytes, e compara ao tamanho de pacote desejado.
        int PacketSizeBytesUTF16;
        if (PacketSize == 1) {
            PacketSizeBytesUTF16 = Packet.getBytes("UTF-16").length - 3;
        } else {
            PacketSizeBytesUTF16 = Packet.getBytes("UTF-16").length - 2;
        }

        System.out.println("Debugger: Measured packet size: " + PacketSizeBytesUTF16);
        if (PacketSizeBytesUTF16 == PacketSize) {
            System.out.println("Debugger: Packet size OK.\n");
        } else {
            System.out.println("Debugger: ERROR: Generated packet has incorrect size.\n");
        }
    }
}
