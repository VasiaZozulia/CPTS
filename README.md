# CPTS Field Manual

Стартовий Obsidian-вольт для підготовки до HackTheBox CPTS і використання під час іспиту.

## Як користуватися
1. Розпакуй цю теку і відкрий її в Obsidian як **Vault** (Open folder as vault).
2. Заповнюй нотатки **своїми словами** під час проходження модулів — не копіюй чітшіти цілком.
3. Кожну команду оформлюй за шаблоном `00-Methodology/_Command-Note-Template.md`.
4. Під час іспиту навігація йде по фазах атаки, а не по модулях.

## Принципи
- **Enumeration — це все.** Іспит не про складність, а про вміння знайти правильне.
- **Один field manual = жодного гуглення під тиском.**
- **Усі креди — в `99-Loot/Credentials.md`.**
- **Звіт = половина успіху.** Веди нотатки так, ніби пишеш звіт уже зараз.

## Карта вольту
- `00-Methodology` — чеклисти, тайм-менеджмент, шаблон нотатки, **Bypasses** (AMSI/AppLocker)
- `01-Recon` → `06-Pivoting` — фази атаки
  - `03-Foothold` містить: Credential-Dumping, NTLM-Relay, Password-Attacks, Shells-Payloads, File-Transfers
  - `04-PrivEsc` містить: Linux і Windows (розширені вектори)
  - `05-Active-Directory` містить: ACL-Abuse, Kerberoast, DCSync, Lateral-Movement, Trusts
- `07-Reporting` — каркас звіту й бібліотека findings
- `99-Loot` — креди, хости, scope
