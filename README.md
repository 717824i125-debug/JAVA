package hemzz;
import java.util.*;

class User {
    private static int idCounter = 1;
    private int userId;
    private String name;
    private String email;
    private String referralCode;
    private int points;

    public User(String name, String email) {
        this.userId = idCounter++;
        this.name = name;
        this.email = email;
        this.referralCode = generateReferralCode();
        this.points = 0;
    }

    private String generateReferralCode() {
        return "REF" + userId + new Random().nextInt(1000);
    }

    public void addPoints(int pts) {
        this.points += pts;
    }

    public boolean deductPoints(int pts) {
        if (pts <= points) {
            this.points -= pts;
            return true;
        }
        return false;
    }

    public int getUserId() { return userId; }
    public String getName() { return name; }
    public String getEmail() { return email; }
    public String getReferralCode() { return referralCode; }
    public int getPoints() { return points; }

    @Override
    public String toString() {
        return name + " (" + points + " pts)";
    }
}

class Referral {
    private static int refCounter = 1;
    private int refId;
    private User inviter;
    private String inviteeEmail;
    private String status; // Pending, SignedUp
    private int bonus;

    public Referral(User inviter, String inviteeEmail) {
        this.refId = refCounter++;
        this.inviter = inviter;
        this.inviteeEmail = inviteeEmail;
        this.status = "Pending";
        this.bonus = 0;
    }

    public void completeSignup(int bonusPoints) {
        this.status = "SignedUp";
        this.bonus = bonusPoints;
        inviter.addPoints(bonusPoints);
    }

    public String getStatus() { return status; }
    public int getBonus() { return bonus; }
    public User getInviter() { return inviter; }
    public String getInviteeEmail() { return inviteeEmail; }
}

class Reward {
    protected static int rewardIdCounter = 1;
    protected int rewardId;
    protected String name;
    protected int pointsRequired;
    protected int inventory;

    public Reward(String name, int pointsRequired, int inventory) {
        this.rewardId = rewardIdCounter++;
        this.name = name;
        this.pointsRequired = pointsRequired;
        this.inventory = inventory;
    }

    public boolean canRedeem(User user) {
        return user.getPoints() >= pointsRequired && inventory > 0;
    }

    public void redeem(User user) {
        if (canRedeem(user) && user.deductPoints(pointsRequired)) {
            consumeInventory();
            deliver(user);
        } else {
            System.out.println("Redemption failed for " + user.getName());
        }
    }

    public void redeem(User user, String promoCode) {
        System.out.println("Redeeming with promo: " + promoCode);
        redeem(user);
    }

    public void redeem(User user, int quantity) {
        for (int i = 0; i < quantity; i++) {
            redeem(user);
        }
    }

    public void consumeInventory() {
        inventory--;
    }

    public void deliver(User user) {
        System.out.println("Reward delivered to " + user.getName());
    }

    public String getName() { return name; }
    public int getRewardId() { return rewardId; }
    public int getInventory() { return inventory; }
}

class VoucherReward extends Reward {
    public VoucherReward(String name, int pointsRequired, int inventory) {
        super(name, pointsRequired, inventory);
    }

    @Override
    public void deliver(User user) {
        System.out.println("Voucher code emailed to " + user.getEmail());
    }
}

class GiftReward extends Reward {
    public GiftReward(String name, int pointsRequired, int inventory) {
        super(name, pointsRequired, inventory);
    }

    @Override
    public void deliver(User user) {
        System.out.println("Physical gift shipped to " + user.getName());
    }
}

class ReferralService {
    private Map<String, User> referralMap = new HashMap<>();
    private List<Referral> referrals = new ArrayList<>();
    private List<Reward> rewards = new ArrayList<>();

    public void registerUser(User user) {
        referralMap.put(user.getReferralCode(), user);
    }

    public void addReward(Reward reward) {
        rewards.add(reward);
    }

    public void trackSignup(String referralCode, String inviteeEmail) {
        User inviter = referralMap.get(referralCode);
        if (inviter != null) {
            Referral ref = new Referral(inviter, inviteeEmail);
            ref.completeSignup(50); // e.g., 50 points for referral
            referrals.add(ref);
            System.out.println("Signup tracked for: " + inviteeEmail);
        } else {
            System.out.println("Invalid referral code.");
        }
    }

    public void redeemRewardById(User user, int rewardId) {
        for (Reward r : rewards) {
            if (r.getRewardId() == rewardId) {
                r.redeem(user);
                return;
            }
        }
        System.out.println("Reward not found.");
    }

    public void redeemRewardByName(User user, String name) {
        for (Reward r : rewards) {
            if (r.getName().equalsIgnoreCase(name)) {
                r.redeem(user);
                return;
            }
        }
        System.out.println("Reward not found.");
    }

    public void printLeaderboard() {
        System.out.println("=== Leaderboard ===");
        referralMap.values().stream()
            .sorted((a, b) -> b.getPoints() - a.getPoints())
            .limit(5)
            .forEach(System.out::println);
    }

    public void printMonthlySummary() {
        System.out.println("=== Monthly Summary ===");
        for (Referral r : referrals) {
            System.out.println("Inviter: " + r.getInviter().getName() +
                " | Invitee: " + r.getInviteeEmail() +
                " | Status: " + r.getStatus() +
                " | Bonus: " + r.getBonus());
        }
    }
}

public class ReferralAppMain {
    public static void main(String[] args) {
        ReferralService service = new ReferralService();

        // Create Users
        User alice = new User("Alice", "alice@example.com");
        User bob = new User("Bob", "bob@example.com");
        User carol = new User("Carol", "carol@example.com");

        service.registerUser(alice);
        service.registerUser(bob);
        service.registerUser(carol);

        // Track Signups
        service.trackSignup(alice.getReferralCode(), "newuser1@example.com");
        service.trackSignup(bob.getReferralCode(), "newuser2@example.com");
        service.trackSignup(alice.getReferralCode(), "newuser3@example.com");

        // Add Rewards
        Reward voucher = new VoucherReward("Amazon Voucher", 60, 5);
        Reward gift = new GiftReward("Coffee Mug", 40, 2);

        service.addReward(voucher);
        service.addReward(gift);

        // Redeem Rewards
        service.redeemRewardById(alice, voucher.getRewardId());
        service.redeemRewardByName(bob, "Coffee Mug");

        // Overloaded redeem
        voucher.redeem(alice, "PROMO10");
        gift.redeem(bob, 1);

        // Reports
        service.printLeaderboard();
        service.printMonthlySummary();
    }
}

