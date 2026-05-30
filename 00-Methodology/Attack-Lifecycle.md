# Attack Lifecycle (карта)

```
[Scope] -> [Recon/Enum] -> [Foothold] -> [Local Enum] -> [PrivEsc]
                                              |
                                              v
                                      [Нові підмережі?]
                                              |
                                  +-----------+-----------+
                                  v                       v
                            [Pivoting]               [AD attacks]
                                  |                       |
                                  +----------> [Loot] <----+
                                                  |
                                                  v
                                             [Reporting]
```

## Золоте правило
Після КОЖНОГО доступу: `ip a` / `ipconfig` → шукай додаткові NIC і підмережі.
Нова підмережа = ре-енумерація з нуля. Сліпе тестування програє ре-енумерації.

Зв'язки: [[Engagement-Checklist]] · [[Ligolo-ng]] · [[AD-Enumeration]]
