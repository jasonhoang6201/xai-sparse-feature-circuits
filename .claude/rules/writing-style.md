# Writing style

Applies to every response and every text I generate, in **any language**:
terminal answers, `.md` and `.html` files, artifacts, docs, commit messages,
code comments, docstrings, identifiers. No per-project exemption.

## 1. Describe the mechanism, not an image of it

Name the thing. To explain something technical, state what calls what, what
runs before or after, what it depends on. Do not substitute a picture for the
mechanism.

**Banned (English):** silver bullet, low-hanging fruit, moving parts, under the
hood, behind the scenes, secret sauce, glue code, magic, dance, journey, heavy
lifting, bread and butter, game changer, the heart/backbone/soul of X, the
missing piece, "think of it as…", "it's like a…", "imagine that…".

**Banned (Vietnamese):** kim chỉ nam, lăng kính, chìa khóa, bức tranh tổng thể,
mắt xích, gốc rễ, xương sống, trái tim của hệ thống, linh hồn, mảnh ghép, đòn
bẩy, cầu nối, bệ phóng, hạt giống, thổi hồn, cánh tay đắc lực, bộ não, "giống
như một…", "hãy tưởng tượng…", "ví như…", "nói nôm na thì…".

The same ban holds in any other language I reply in — these two lists are
examples of the category, not its full extent.

**Use instead** — words for logical relations: cause, precondition, basis,
consequence, depends on, calls, returns, overrides, runs before/after, blocks,
caches, initialization order. (VN: nguyên nhân, điều kiện, tiền đề, cơ sở, hệ
quả, phụ thuộc vào, gọi tới, trả về, ghi đè, chạy trước/sau, chặn.)

**Exception — read this before applying rule 1.** Standard technical terms with
pictorial roots are names, not figures of speech. Use them normally. Avoiding
them and paraphrasing instead is a worse failure than a metaphor:

> branch, tree, root, leaf, node, parent/child, stack, heap, queue, deadlock,
> race condition, bottleneck, memory leak, hook, wrapper, pipeline, thread,
> pool, bucket, mount, socket, handshake, garbage collector, fork, merge,
> rollback, snapshot, migration, dependency, header, footer, payload, endpoint,
> middleware, dashboard, container, image, cluster.

Never write "cấu trúc phân nhánh dạng cây" for *branch*, or "a state where two
processes each hold what the other needs" for *deadlock*.

### Wrong → right

| Wrong | Right |
|---|---|
| "Cache is like a notebook that helps the system remember faster." | "Cache stores the previous call's result, so the next call skips the DB query." |
| "This file is the heart of the auth module." | "This file holds all token issuing and validation logic." |
| "Middleware bridges the request and the handler." | "Middleware runs before the handler and can modify or block the request." |
| "Under the hood, React handles this for you." | "React diffs the old and new virtual DOM and rewrites only the changed nodes." |
| "The last missing piece of the system." | "The last component not yet implemented." |
| "Cache như một cuốn sổ tay giúp hệ thống nhớ nhanh hơn." | "Cache lưu kết quả của lần gọi trước, nên lần sau không truy vấn DB." |
| "File này là trái tim của module auth." | "File này chứa toàn bộ logic cấp và kiểm tra token." |

## 2. No empty praise adjectives

Drop *powerful, robust, seamless, elegant, blazing fast, incredibly important,
absolutely critical, beautiful*; VN: *cực kỳ mạnh mẽ, vô cùng quan trọng, rất
tuyệt vời, siêu nhanh, đỉnh cao, hoàn hảo* — unless a number follows.

Replace with a measurement, or with the concrete consequence if no number exists.

| Wrong | Right |
|---|---|
| "This library is extremely fast." | "It parses 10MB of JSON in ~80ms, about 3× faster than `JSON.parse`." |
| "Validating input is critically important." | "Without validation the query receives an empty string and returns the whole table." |
| "Refactor này giúp code sạch hơn rất nhiều." | "Refactor này bỏ 3 nhánh if lồng nhau, hàm còn 1 điểm return." |

## 3. Generated files (HTML / Markdown / docs)

Rules 1 and 2 apply, plus:

- Headings state content, not feeling: "Token issuing flow", not "A journey
  through tokens".
- No warm-up opener ("In today's fast-moving world of software…"). First
  sentence carries content.
- No "Conclusion" section that only restates what was just written.
- Image and diagram captions describe what is shown, not how to feel about it.

## 4. Self-check before output

1. Any banned phrase, in any language? Rewrite that sentence as a mechanism.
2. Any praise adjective without a number or consequence? Cut or replace it.
3. Any standard term from the Exception list dodged and paraphrased? Put the
   term back.

# Output files

Never publish an Artifact (claude.ai hosted page) unless I explicitly say
"artifact". When I ask for HTML, write a normal local `.html` file in the
working directory and give me its path. Do not call the Artifact tool's
quickstart or publish for it.
