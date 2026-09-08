"use client";

import { useMemo, useState } from "react";
import { computeQuote, DEFAULTS, formatNum } from "@/lib/quote";

type Props = {
  priceKrwPerGram: number;
  asOf: string;
};

const KARATS = [24, 22, 18, 14, 10];

export default function QuoteTool({
  priceKrwPerGram,
  asOf,
}: Props) {
  const [name, setName] = useState("");
  const [phone, setPhone] = useState("");
  const [karat, setKarat] = useState(18);
  const [weight, setWeight] = useState("");
  const [condition, setCondition] = useState<
    "good" | "normal" | "damaged"
  >("good");
  const [askingPrice, setAskingPrice] = useState("");
  const [submitted, setSubmitted] = useState(false);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState("");

  const weightG = Number(weight) || 0;
  const askingKrw = Number(
    askingPrice.replace(/[^0-9]/g, "")
  ) || 0;

  const q = useMemo(() => {
    if (weightG <= 0) return null;

    return computeQuote({
      priceKrwPerGram,
      weightG,
      karat,
      condition,
      marginPct: DEFAULTS.marginPct,
      askingPriceKrw: askingKrw || null,
    });
  }, [
    priceKrwPerGram,
    weightG,
    karat,
    condition,
    askingKrw,
  ]);

  async function submitRequest() {
    setError("");

    if (!name.trim()) {
      setError("이름을 입력해주세요.");
      return;
    }

    if (!phone.trim()) {
      setError("연락처를 입력해주세요.");
      return;
    }

    if (!weightG || weightG <= 0) {
      setError("중량을 입력해주세요.");
      return;
    }

    setLoading(true);

    try {
      const response = await fetch(
        "/api/quote-request",
        {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({
            name: name.trim(),
            phone: phone.trim(),
            karat,
            weightG,
            condition,
            askingPriceKrw:
              askingKrw || null,
            estimatedOffer: q?.maxOffer ?? null,
            asOf,
          }),
        }
      );

      const result = await response.json();

      if (!response.ok) {
        throw new Error(
          result?.error ||
            "견적 상담 신청에 실패했습니다."
        );
      }

      setSubmitted(true);
    } catch (err) {
      setError(
        err instanceof Error
          ? err.message
          : "견적 상담 신청에 실패했습니다."
      );
    } finally {
      setLoading(false);
    }
  }

  if (submitted) {
    return (
      <div className="card card-gold quote-success">
        <div className="eyebrow">상담 신청 완료</div>

        <h2
          className="serif"
          style={{
            fontSize: 30,
            margin: "8px 0 10px",
          }}
        >
          견적 상담 신청이 완료되었습니다.
        </h2>

        <p
          className="muted"
          style={{
            lineHeight: 1.8,
            marginBottom: 18,
          }}
        >
          입력해주신 연락처로 상담을 진행해드립니다.
          <br />
          최종 매입가는 상품의 실제 상태와 순도 확인
          후 확정됩니다.
        </p>

        <button
          type="button"
          className="btn btn-gold"
          onClick={() => {
            setSubmitted(false);
            setError("");
          }}
        >
          새로운 견적 신청하기
        </button>
      </div>
    );
  }

  return (
    <div>
      <div className="card" style={{ padding: 20 }}>
        <div className="eyebrow">
          GOLD QUOTE
        </div>

        <h2
          className="serif"
          style={{
            fontSize: 30,
            margin: "6px 0 8px",
          }}
        >
          금 매입 견적
        </h2>

        <p
          className="muted"
          style={{
            margin: "0 0 20px",
            lineHeight: 1.7,
          }}
        >
          금의 종류와 중량을 입력하시면 오늘 시세를
          기준으로 예상 매입가를 확인할 수 있습니다.
        </p>

        <div className="form-grid">
          <label className="field">
            <span>이름</span>

            <input
              value={name}
              onChange={(e) =>
                setName(e.target.value)
              }
              placeholder="홍길동"
            />
          </label>

          <label className="field">
            <span>연락처</span>

            <input
              value={phone}
              onChange={(e) =>
                setPhone(e.target.value)
              }
              placeholder="010-0000-0000"
              inputMode="tel"
            />
          </label>

          <label className="field">
            <span>순도</span>

            <select
              value={karat}
              onChange={(e) =>
                setKarat(Number(e.target.value))
              }
            >
              {KARATS.map((k) => (
                <option key={k} value={k}>
                  {k}K
                  {k === 24
                    ? " · 순금"
                    : ""}
                </option>
              ))}
            </select>
          </label>

          <label className="field">
            <span>중량(g)</span>

            <input
              value={weight}
              onChange={(e) =>
                setWeight(
                  e.target.value.replace(
                    /[^0-9.]/g,
                    ""
                  )
                )
              }
              placeholder="예: 10.5"
              inputMode="decimal"
            />
          </label>

          <label className="field">
            <span>상품 상태</span>

            <select
              value={condition}
              onChange={(e) =>
                setCondition(
                  e.target.value as
                    | "good"
                    | "normal"
                    | "damaged"
                )
              }
            >
              <option value="good">
                양호
              </option>

              <option value="normal">
                보통
              </option>

              <option value="damaged">
                손상 있음
              </option>
            </select>
          </label>

          <label className="field">
            <span>
              희망 매입가{" "}
              <em className="muted">
                (선택)
              </em>
            </span>

            <input
              value={askingPrice}
              onChange={(e) =>
                setAskingPrice(
                  e.target.value.replace(
                    /[^0-9]/g,
                    ""
                  )
                )
              }
              placeholder="예: 1,000,000"
              inputMode="numeric"
            />
          </label>
        </div>

        {q && (
          <div
            className="quote-result"
            style={{
              marginTop: 22,
            }}
          >
            <div className="quote-result-head">
              <span>예상 매입가</span>

              <strong className="num">
                {formatNum(q.maxOffer)}원
              </strong>
            </div>

            <div className="quote-details">
              <div>
                <span>순금 환산 중량</span>

                <b className="num">
                  {formatNum(
                    q.pureGoldWeightG,
                    2
                  )}
                  g
                </b>
              </div>

              <div>
                <span>정제 손실 반영</span>

                <b className="num">
                  {formatNum(
                    q.refiningLossPct,
                    1
                  )}
                  %
                </b>
              </div>

              <div>
                <span>기준 시세</span>

                <b className="num">
                  {formatNum(
                    priceKrwPerGram
                  )}
                  원/g
                </b>
              </div>
            </div>

            <p
              className="muted"
              style={{
                margin: "12px 0 0",
                fontSize: 12,
                lineHeight: 1.6,
              }}
            >
              위 금액은 입력하신 정보를 기준으로 한
              참고 금액입니다. 실제 매입가는 실물의
              순도·중량·상태 확인 후 최종 확정됩니다.
            </p>
          </div>
        )}

        {error && (
          <div
            className="verdict-reject"
            style={{
              marginTop: 14,
              fontSize: 14,
            }}
          >
            {error}
          </div>
        )}

        <button
          type="button"
          className="btn btn-gold"
          style={{
            width: "100%",
            marginTop: 20,
            minHeight: 52,
            fontSize: 16,
          }}
          disabled={loading}
          onClick={submitRequest}
        >
          {loading
            ? "신청 중..."
            : "견적 상담 신청하기"}
        </button>

        <p
          className="muted"
          style={{
            fontSize: 12,
            lineHeight: 1.6,
            margin: "10px 0 0",
            textAlign: "center",
          }}
        >
          상담을 위해 입력하신 연락처가 상담 목적으로
          이용될 수 있습니다.
        </p>
      </div>
    </div>
  );
}