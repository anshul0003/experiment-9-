import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;
import org.hibernate.Session;
import org.hibernate.Transaction as HibernateTransaction;
import jakarta.persistence.*;
import java.util.Date;

@Entity
class Account {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;
    private String name;
    private double balance;

    public int getId() { return id; }
    public void setId(int id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public double getBalance() { return balance; }
    public void setBalance(double balance) { this.balance = balance; }
}

@Entity
class Transaction {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;
    private int fromAccountId;
    private int toAccountId;
    private double amount;
    @Temporal(TemporalType.TIMESTAMP)
    private Date timestamp = new Date();

    public Transaction() {}
    public Transaction(int from, int to, double amount) {
        this.fromAccountId = from;
        this.toAccountId = to;
        this.amount = amount;
    }
}

public class BankApp {
    public static void main(String[] args) {
        SessionFactory factory = new Configuration()
                .configure("hibernate.cfg.xml")
                .addAnnotatedClass(Account.class)
                .addAnnotatedClass(Transaction.class)
                .buildSessionFactory();

        transferMoney(factory, 1, 2, 500.0);
        transferMoney(factory, 1, 2, 100000.0);

        factory.close();
    }

    public static void transferMoney(SessionFactory factory, int fromId, int toId, double amount) {
        Session session = factory.getCurrentSession();
        HibernateTransaction tx = null;

        try {
            tx = session.beginTransaction();

            Account from = session.get(Account.class, fromId);
            Account to = session.get(Account.class, toId);

            if (from.getBalance() < amount) throw new RuntimeException("Insufficient funds");

            from.setBalance(from.getBalance() - amount);
            to.setBalance(to.getBalance() + amount);

            Transaction txn = new Transaction(fromId, toId, amount);
            session.save(txn);

            tx.commit();
            System.out.println("Transaction successful");
        } catch (Exception e) {
            if (tx != null) tx.rollback();
            System.out.println("Transaction failed: " + e.getMessage());
        } finally {
            session.close();
        }
    }
}
