---
description: 此功能採用了鏈式dsl設計編輯/新增物品。
---

# 物品編輯

{% hint style="info" %}
你可以直接參閱 [Javadocs](https://eric2788.github.io/ELDependenci/) 來查看教學。
{% endhint %}

```java
@Commander(
        name = "test",
        description = "test command",
        alias = {"tes", "te"},
        playerOnly = true
)
public class TestCommand implements CommandNode {

    @Inject
    private ItemStackService service;

    @Override
    public void execute(CommandSender commandSender) {
        var player = (Player)commandSender;
        var stone = service.build(Material.STONE)
                .amount(5)
                .durability(10)
                .editItemMeta(meta -> meta.removeItemFlags(ItemFlag.HIDE_UNBREAKABLE))
                .display("&aGOOD STONE!")
                .lore("&cgood", "&astone")
                .editPersisData(data -> data.set(NamespacedKey.minecraft("for.what"), PersistentDataType.STRING, "huh"))
                .onInteractEventTemp(e -> e.getPlayer().sendMessage("you "+e.getAction().name()+" the stone!"))
                .getItem();
        player.getInventory().addItem(stone);

        var handItem = service.edit(player.getInventory().getItemInMainHand())
                                                    .lore(l -> l.add("&cthis item is in your mainhand"))
                                                    .getItem();
        player.getInventory().setItemInMainHand(handItem);
    }

}
```

