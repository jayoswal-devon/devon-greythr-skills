## About this repository
This respository contains **skills for agents** to perform Day2Day tasks like swipe-in/out, attendance regularization, leaves check, leaves apply etc on the [DevOn's Greythr Portal](https://devon.greythr.com) 

## Instructions to use the skills

- **All Skills**
```bash
npx skills add jayoswal-devon/devon-greythr-skills
```

- **Browser-Act Skills** [Required & automatically fetched by other skills]'

```bash
npx skills add https://github.com/jayoswal-devon/devon-greythr-skills --skill browser-act
```

- **Task Specific Skills** e.g. swipe-in skill 

```bash
npx skills add https://github.com/jayoswal-devon/devon-greythr-skills --skill swipe-in
```

- available skills are `swipe-in`, `swipe-out`, `check-leaves-balances`

## Skills Set
### 1. check-leaves-balances
- Check available leaves balances on Greythr portal across all leave types
### 2. swipe-in
- Swipe-in to mark attendance on Greythr portal
### 3. swipe-out
- Swipe-out to mark attendance on Greythr portal

## Token Count
| Skill Name | Token Count |
|------------|-------------|
| browser-act | ~ 5,300 |
| check-leaves-balances | ~ 600 |
| swipe-in | TODO |
| swipe-out | TODO |

_Used [OpenAI Tokenizer](https://platform.openai.com/tokenizer)_
