# 연관관계 편의 메소드

엔티티 매핑도중에 어느 한쪽이 다수가 되면 이에 연관관계 편의 메소드가 늘 있어야 합니다.

할 때마다 헷갈려서 한번 정리하고자 하는 의미입니다.

### N:1

멤버가 `setTeam()` 으로 팀을 추가할 때 해당 팀에 본인이 없으면 해당 팀에 본인을 추가합니다.

```java
public class Member {
    
    private Team team;
		//연관관계 편의 메소드
    public void setTeam(Team team) {
        this.team = team;
        if(!team.getMembers().contains(this)) {
            team.getMembers().add(this);
        }
    }
}

public class Team {

    private List<Member> members = new ArrayList<Member>();
		//연관관계 편의 메소드
    public void addMember(Member member) {
        this.members.add(member);
        if(member.getTeam() != this) {
            member.setTeam(this);
        }
    }
}
```

### N:M

회원이 물품을 구매할 때 회원의 물품목록에 `addProduct()` 를 하면서 해당 물품의 멤버 리스트에 본인이 없으면 본인을 추가합니다